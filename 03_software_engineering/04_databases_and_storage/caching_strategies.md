---
layer: 03_software_engineering
type: engineering
tool: Redis/Memcached
status: growing
tags: [caching, redis, performance, distributed-systems, invalidation]
created: 2026-03-05
---

# Caching Strategies

## Purpose

Reference for caching patterns, eviction policies, invalidation strategies, and distributed
caching considerations. Caching is one of the highest-leverage performance tools but also
a common source of subtle bugs when invalidation is wrong.

## Architecture

### Cache-Aside (Lazy Loading)

The most common pattern. Application code manages the cache explicitly.

```python
def get_user(user_id: int) -> dict:
    key = f"user:{user_id}"
    cached = redis.get(key)
    if cached:
        return json.loads(cached)          # cache hit

    user = db.query("SELECT * FROM users WHERE id = %s", user_id)  # cache miss
    redis.setex(key, 300, json.dumps(user))  # store with 5-min TTL
    return user

def update_user(user_id: int, data: dict):
    db.execute("UPDATE users SET ... WHERE id = %s", user_id)
    redis.delete(f"user:{user_id}")        # invalidate on write
```

**Characteristics:**
- Cache is only populated on demand — no wasted cache space for cold data.
- First request after expiry is always slow (cache miss penalty).
- Application code must handle cache-miss path; adds complexity.
- Risk: **stale data** if write path forgets to invalidate.

---

### Read-Through

Cache sits between application and database; cache fetches from DB on miss automatically.
Common in ORM-level caches (Hibernate second-level cache, ActiveRecord cache stores).

```
App → Cache → DB   (on miss: Cache fetches from DB, stores, returns)
App → Cache        (on hit: Cache returns directly)
```

- Transparent to application; cache logic centralised.
- First-time fetch is still slow (same cold-start as cache-aside).
- Cache must understand the data source — tighter coupling.

---

### Write-Through

Every write goes to cache AND database synchronously before acknowledging the client.

```python
def update_product(product_id, price):
    db.execute("UPDATE products SET price=%s WHERE id=%s", price, product_id)
    redis.setex(f"product:{product_id}:price", 3600, price)  # write to cache too
    return "ok"
```

- Cache is always consistent with DB at the cost of write latency (two writes).
- Works best when read/write ratio is high and cache hit rate matters.
- Wastes memory: items are cached even if never read again.

---

### Write-Back (Write-Behind)

Application writes to cache only; a background process asynchronously flushes to DB.

```
App → Cache   (fast ack)
Background worker → DB   (async, e.g., every 500ms or on eviction)
```

- Lowest write latency of all patterns.
- **Risk**: data loss if cache crashes before flush — acceptable for analytics,
  not for financial transactions.
- Useful for high-frequency counters (page views, likes) — batch flush to DB.

---

### Write-Around

Writes go directly to DB, bypassing cache. Cache is populated only on subsequent reads.

- Good when written data is unlikely to be read soon (bulk imports, log data).
- Avoids cache pollution but guarantees cache miss on first read.

---

## Implementation Notes

### Eviction Policies

When the cache is full, Redis must evict entries. Set via `maxmemory-policy` in redis.conf.

| Policy | Behaviour | Best for |
|---|---|---|
| `noeviction` | Return error on write when full | Never use in cache role |
| `allkeys-lru` | Evict least-recently-used across all keys | General-purpose cache |
| `volatile-lru` | LRU among keys with TTL set | Mixed TTL / permanent data |
| `allkeys-lfu` | Evict least-frequently-used across all keys | Skewed access distribution |
| `volatile-lfu` | LFU among keys with TTL | Mixed workloads |
| `volatile-ttl` | Evict key closest to expiry | Prefer removing stale data first |
| `allkeys-random` | Random eviction | Rarely useful |

```
# redis.conf
maxmemory 2gb
maxmemory-policy allkeys-lru
```

**LRU vs LFU:**
- LRU is better when access recency predicts future access (most web caches).
- LFU is better when certain items are accessed frequently regardless of recency
  (viral content, popular products).

**TTL as an eviction strategy:**
```python
# Always set a TTL — prevents unbounded growth and acts as a safety net
# for stale data even if explicit invalidation is missed
redis.setex(key, ttl_seconds, value)
```

---

### Cache Invalidation Strategies

*"There are only two hard things in computer science: cache invalidation and naming things."*

**1. TTL-based (time-to-live):**  
Simplest. Set an expiry; accept bounded staleness. Good for data that changes slowly
or where slightly stale data is acceptable (product catalog, exchange rates).

**2. Event-driven invalidation:**  
Write path publishes an event; cache subscribers delete or update affected keys.
```python
# Publisher (in write path)
redis.publish("invalidate:product", product_id)

# Subscriber (cache invalidation worker)
p = redis.pubsub()
p.subscribe("invalidate:product")
for msg in p.listen():
    redis.delete(f"product:{msg['data']}")
```

**3. Versioned keys (cache busting):**  
Embed a version in the key. Changing the version effectively invalidates all old keys.
```python
CACHE_VERSION = "v3"

def cache_key(resource_id):
    return f"{CACHE_VERSION}:product:{resource_id}"
```
Old keys are left to expire via TTL. Simple, no invalidation coordination needed.
Drawback: old keys accumulate until TTL expires — monitor memory.

**4. Write-through + delete-on-write (most reliable):**  
Update DB, then `DEL` cache key. Next read repopulates. Simpler than updating the
cached value (avoids race between concurrent writers).

---

### Thundering Herd Problem

When a popular cached item expires, many concurrent requests miss the cache simultaneously
and all hit the database at once.

**Solution 1: Mutex / distributed lock**
```python
import redis_lock

def get_expensive(key):
    val = redis.get(key)
    if val:
        return val

    with redis_lock.Lock(redis, f"lock:{key}", expire=5):
        # Double-check after acquiring lock
        val = redis.get(key)
        if val:
            return val
        val = expensive_db_query()
        redis.setex(key, 300, val)
    return val
```

**Solution 2: Probabilistic Early Expiration (PER)**  
Jitter the effective TTL so different instances re-fetch at slightly different times.
```python
import math, random, time

def fetch_with_per(key, ttl, beta=1.0):
    result = redis.get(key)        # returns (value, remaining_ttl) if stored with expiry
    if result is None:
        return recompute_and_store(key, ttl)

    value, remaining = result
    # Recompute early with probability proportional to recomputation time
    if -beta * recompute_time * math.log(random.random()) >= remaining:
        return recompute_and_store(key, ttl)
    return value
```

**Solution 3: Stale-while-revalidate**  
Serve stale data while refreshing asynchronously in the background.
Common in HTTP caches (`Cache-Control: stale-while-revalidate=60`).

---

### Redis Data Structures Quick Reference

```python
# String — scalar values, counters, serialised JSON
redis.set("key", "value", ex=60)
redis.incr("counter")
redis.incrby("counter", 5)

# Hash — object fields; memory-efficient for small objects
redis.hset("user:1", mapping={"name": "Ada", "score": 100})
redis.hincrby("user:1", "score", 10)
redis.hgetall("user:1")

# List — queues, stacks, recent activity log
redis.rpush("queue", "task1", "task2")   # enqueue
redis.blpop("queue", timeout=0)          # blocking dequeue (worker)
redis.lrange("feed:user:1", 0, 49)       # last 50 feed items

# Set — tags, unique visitors, deduplication
redis.sadd("online_users", "user:1", "user:2")
redis.scard("online_users")              # count
redis.srandmember("online_users", 3)     # random sample

# Sorted Set — leaderboards, rate limiting, priority queues
redis.zadd("scores", {"alice": 9800.0, "bob": 9200.0})
redis.zrevrangebyscore("scores", "+inf", "-inf", start=0, num=10)
redis.zincrby("scores", 100, "alice")

# Stream — append-only event log; consumer groups
redis.xadd("events:clicks", {"user_id": "42", "page": "/home"})
redis.xreadgroup("mygroup", "worker-1", {"events:clicks": ">"}, count=100)
```

---

### Distributed Caching Considerations

**Consistent Hashing:**
Partitions key space across nodes; adding/removing a node only remaps ~1/N keys.
Libraries: `uhashring` (Python), native in Redis Cluster.

**Redis Cluster:**
- Automatically shards across 16384 hash slots.
- Multi-key operations (MGET, pipeline) must target the same slot — use hash tags `{user:42}`.
- No cross-slot transactions.

**Read replicas:**
```
redis.StrictRedis(host="replica-1")  # direct replica reads for non-critical paths
```

**Cache stampede across services:**
In microservice architectures, different services may cache the same backing data.
Use a shared cache tier (Redis) and a consistent invalidation event bus (Kafka, Redis Pub/Sub).

**Cache warming:**
```python
# Pre-populate cache before traffic hits — prevents cold-start stampede
def warm_cache():
    popular_ids = db.query("SELECT id FROM products ORDER BY views DESC LIMIT 1000")
    for pid in popular_ids:
        get_product(pid)   # normal read-through warms cache as side effect
```

## Trade-offs

| Strategy | Consistency | Write Latency | Complexity |
|---|---|---|---|
| Cache-aside | Eventual | Normal | Low |
| Read-through | Eventual | Normal | Low–Medium |
| Write-through | Strong | Higher (+cache write) | Medium |
| Write-back | Weak (data loss risk) | Lowest | High |
| Write-around | Eventual | Normal | Low |

**Key decisions:**
- **TTL length**: short TTL → more DB load, less stale data; long TTL → inverse.
- **In-process vs external cache**: in-process (e.g., `functools.lru_cache`) is fastest
  but not shared across instances; Redis is shared but adds network latency (~0.5 ms).
- **Serialisation format**: MessagePack / protobuf over JSON for high-throughput caches
  (faster ser/de, smaller payloads).

## References

- [Redis documentation — Data types](https://redis.io/docs/data-types/)
- [Redis documentation — Eviction policies](https://redis.io/docs/manual/eviction/)
- [Thundering Herd — optimal probabilistic cache stampede prevention](https://cseweb.ucsd.edu/~avattani/papers/cache_stampede.pdf)
- *Designing Data-Intensive Applications* — Martin Kleppmann (Ch. 5)
- [AWS ElastiCache Caching Strategies](https://docs.aws.amazon.com/AmazonElastiCache/latest/mem-ug/Strategies.html)

## Links
- [[nosql_patterns|NoSQL Patterns]]
- [[scalability_patterns|Scalability Patterns]]
- [[distributed_systems|Distributed Systems]]
- [[05_ai_engineering/03_rag_and_agents/vector_stores|Vector Stores]]
