---
layer: 03_software_engineering
type: concept
status: growing
tags: [scalability, sharding, caching, rate-limiting, replication, stateless]
created: 2026-03-05
---

# Scalability Patterns

## Definition

Scalability is a system's ability to handle increasing load without degrading performance beyond acceptable bounds. Scalability patterns are architectural decisions and algorithms that enable a system to grow — more users, more data, more requests — while preserving low latency and high availability.

## Intuition

You scale a system by either making each component more powerful (vertical) or by adding more components (horizontal). Horizontal scaling is preferable at cloud scale because commodity hardware is cheaper than specialized hardware and it enables redundancy. The prerequisite for horizontal scaling is that components are stateless or that state is managed externally and partitioned effectively.

## Formal Description

### Horizontal vs Vertical Scaling

**Vertical scaling (scale-up)**
- Replace a node with a larger one: more CPU cores, RAM, faster disk
- Requires downtime for hardware changes (unless live migration is possible)
- Has a hard physical ceiling
- Simple: no distributed coordination required
- Appropriate for: databases in early stage, legacy single-threaded applications

**Horizontal scaling (scale-out)**
- Add more nodes of the same size to a pool
- Requires workload distribution (load balancer, consistent hashing, partitioning)
- Theoretically unbounded; adds redundancy automatically
- Appropriate for: stateless web/app tier, caches, distributed databases, queues
- Challenge: distributed state — sessions, locks, aggregations must be externalized

### Stateless Services

A stateless service holds no client-specific state in application memory. All state lives in an external store (database, Redis, object storage). Any instance can handle any request.

Benefits:
- Horizontal scaling is trivial: add instances behind a load balancer
- Rolling deploys with zero downtime: drain old instances, spin up new ones
- No sticky sessions required

Implementation:
- Store session tokens in Redis or JWTs (self-contained, signed)
- Store uploaded files in S3 / object storage, not local disk
- Externalize in-memory caches to Redis or Memcached
- Configuration via environment variables or secret stores (not local files)

### Read Replicas

A primary database handles all writes; one or more replicas receive an asynchronous stream of changes and serve read traffic.

- **Replication lag**: replicas trail primary by milliseconds to seconds depending on write volume and network
- **Read-your-writes problem**: a user writes then immediately reads; if routed to a lagging replica they see stale data. Fix: route a user's reads to primary for a short window after a write, or use synchronous replication for critical paths
- **Scaling reads**: typical OLTP workloads are 80–95% reads. Read replicas can multiply read capacity almost linearly
- **Failover**: if primary fails, promote a replica. Semi-synchronous replication reduces data loss risk

When read replicas are insufficient (write-heavy workload), move to sharding.

### Sharding Strategies

Sharding (horizontal partitioning) divides data across multiple database nodes, each owning a subset of the keyspace.

**Range sharding**
- Each shard owns a contiguous range of keys (e.g., user IDs 0–1M, 1M–2M)
- Natural for range queries: scan a single shard
- Risk: hot partitions if data is not uniformly distributed (e.g., all new users go to the last shard)
- Solution: pre-split ranges, dynamic range splitting (used in HBase, Google Bigtable)

**Hash sharding**
- Shard = hash(key) mod N
- Uniform distribution of keys across shards (assuming a good hash function)
- Range queries are inefficient: must scatter-gather across all shards
- Problem: resharding requires moving ~half of all keys when N changes

**Consistent hashing**
- Keys and nodes are mapped onto a ring by hash value
- Each key is owned by the first node clockwise on the ring
- Adding/removing a node affects only its immediate neighbor's keys (~1/N fraction)
- Virtual nodes (vnodes): each physical node owns multiple positions on the ring, improving balance
- Used in: Amazon DynamoDB, Cassandra, Apache Riak, Akamai CDN

**Directory-based sharding**
- A lookup service maps each key (or key range) to its shard
- Maximum flexibility: any partition scheme, easy resharding (update the directory)
- The directory itself becomes a bottleneck / SPOF; must be cached and replicated

**Sharding challenges**:
- Cross-shard joins require scatter-gather or application-level JOINs
- Cross-shard transactions require distributed transaction protocols (2PC, sagas)
- Re-sharding (changing N) is operationally complex; plan ahead for growth

### Fan-Out Patterns

**Fan-out on write (push model)**
- When a user creates content, immediately push it to all followers' feeds/inboxes
- Write amplification: a celebrity with 50M followers creates 50M write operations
- Reads are O(1): just read the pre-computed feed
- Appropriate for users with small/medium follower counts
- Used by: Twitter (for non-celebrities), Facebook

**Fan-out on read (pull model)**
- Each user's feed is computed at read time by fetching posts from followed accounts and merging
- No write amplification; scales well for celebrities
- Expensive reads: O(followers) queries per feed load, requires heavy caching
- Used for: celebrity accounts (Twitter's hybrid model)

**Hybrid approach (Twitter)**
- Fan-out on write for users with < N followers (say, 1M)
- Fan-out on read for celebrities (> N followers); results merged at read time

### Rate Limiting Algorithms

Rate limiting protects services from abuse and ensures fair resource sharing.

**Token bucket**
- A bucket holds up to `capacity` tokens; tokens are added at rate `r` tokens/second
- Each request consumes one token; if bucket is empty, request is rejected or queued
- Allows bursts up to `capacity`; sustained rate cannot exceed `r`
- Easy to implement in Redis with a counter and timestamp
- Used by: AWS API Gateway, Stripe

**Leaky bucket**
- Requests enter a queue (the bucket); they are processed at a fixed output rate
- Excess requests overflow (are dropped)
- Enforces a strict output rate; no burst allowed
- Suitable for smoothing traffic to a downstream service with a fixed capacity

**Fixed window counter**
- Count requests in the current time window (e.g., per minute)
- Problem: a user can send 2× the limit by placing requests at the end of one window and the start of the next
- Simple and memory-efficient

**Sliding window log**
- Store timestamps of all requests in a sorted set (e.g., Redis ZSET)
- On each request, remove entries older than window, count remaining, add current timestamp
- Accurate but memory-intensive for high-rate users

**Sliding window counter (approximate)**
- Combine current and previous window counters weighted by position in the current window
- `count = prev_count × (1 - elapsed/window) + curr_count`
- Accurate enough for most purposes; O(1) memory per user

Distributed rate limiting requires atomic operations. Redis `INCR` + `EXPIRE` for simple cases; Lua scripts for compound operations; Redis Cluster for multi-shard scenarios.

### Caching Strategies

**Write-through**
- Write goes to cache and database synchronously before acknowledging to the client
- Cache always has fresh data; no stale reads
- Higher write latency (two writes per operation)
- Best when reads follow writes closely and stale data is unacceptable

**Write-back (write-behind)**
- Write acknowledged after writing to cache only; database updated asynchronously
- Very low write latency
- Risk of data loss if cache node fails before flush
- Best for write-heavy workloads where some loss is tolerable (session data, counters)

**Write-around**
- Write goes directly to database, bypassing cache
- Cache is populated only on subsequent reads (cache-aside)
- Good when data is written once and rarely read (large files, audit logs)

**Cache-aside (lazy loading)**
- Application checks cache; on miss, reads from DB, populates cache with TTL
- Cache holds only data that has been requested at least once
- Risk: cold start performance, thundering herd on popular keys expiring simultaneously
- Mitigation: randomize TTLs, use "dog-pile" prevention (single-flight, mutex locks)

**Read-through**
- Cache sits transparently in front of DB; on miss, cache fetches from DB and returns result
- Application only talks to cache
- Simplifies application code; cache provider handles invalidation logic

**Cache invalidation strategies**:
- TTL (time-to-live): simplest; accepts bounded staleness
- Event-driven invalidation: DB triggers or application events explicitly delete/update cache keys
- Versioned keys: `user:123:v42`; increment version on update; old keys expire naturally
- Cache-aside with write invalidation: on write to DB, delete cache key (not update — avoids race condition)

## Applications

- URL shortener: consistent hashing across redirect servers; Redis cache for hot URLs
- Social media feed: fan-out on write + read replicas + Redis sorted set per user
- E-commerce flash sale: token bucket rate limiting + write-through cache for inventory
- Multi-region deployment: geographically distributed shards + eventual consistency across regions
- Search service: sharding by document ID, read replicas for search load

## Trade-offs

| Pattern | Pro | Con |
|---------|-----|-----|
| Horizontal scaling | Redundancy, unbounded growth | State externalization, coordination overhead |
| Hash sharding | Uniform distribution | No range queries, resharding complexity |
| Fan-out on write | O(1) reads | Write amplification for popular users |
| Token bucket | Burst-friendly | Slightly complex to implement atomically |
| Write-through cache | Strong consistency | Higher write latency |
| Write-back cache | Low write latency | Data loss risk |

## Links
- [[distributed_systems|Distributed Systems]]
- [[caching_strategies|Caching]]
- [[nosql_patterns|NoSQL Patterns]]
- [[kubernetes_basics|Kubernetes]]
- [[04_ml_engineering/05_deployment_and_serving/serving_patterns|Model Serving Patterns]]
