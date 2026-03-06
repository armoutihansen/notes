---
layer: 03_software_engineering
type: concept
status: growing
tags: [system-design, architecture, scalability, databases, caching]
created: 2026-03-05
---

# System Design Fundamentals

## Definition

System design is the process of defining architecture, components, interfaces, and data flows to satisfy specified requirements. In an interview or production context it encompasses both the decomposition of a problem into services and the selection of appropriate infrastructure primitives (load balancers, caches, queues, databases) to meet scale and reliability targets.

## Intuition

Every system design decision is a trade-off between competing constraints: consistency vs. availability, cost vs. latency, simplicity vs. flexibility. The goal is not to find a perfect design—it does not exist—but to articulate which trade-offs are acceptable given the requirements and to build a system that can evolve when those requirements change.

## Formal Description

### Requirements Gathering

**Functional requirements** define *what* the system does:
- User-facing features (create post, send message, search)
- Data ingestion and processing pipelines
- External integrations and APIs

**Non-functional requirements** define *how well* it does it:
- **Latency**: p50/p99 read/write targets (e.g., < 100 ms p99 read)
- **Throughput**: requests per second, messages per second
- **Availability**: target uptime SLA (99.9% = 8.7 h/year downtime; 99.99% = 52 min/year)
- **Durability**: tolerance for data loss (RPO)
- **Consistency**: whether all users see the same data at the same time
- **Scalability**: expected growth (10x in 2 years?)

### Capacity Estimation (Back-of-Envelope)

Useful constants to memorize:
| Unit | Value |
|------|-------|
| 1 million requests/day | ~12 req/s |
| 1 billion requests/day | ~12,000 req/s |
| 1 KB × 1M users | ~1 GB |
| HDD read | ~100 MB/s |
| SSD read | ~500 MB/s–3 GB/s |
| Network (datacenter) | ~10 Gbps |
| RTT same region | ~1 ms |
| RTT cross-continent | ~100–150 ms |

**Storage estimation example** (Twitter-scale):
- 500 M tweets/day × 300 bytes/tweet = 150 GB/day raw text
- Over 5 years = ~270 TB
- With replication (3×) and indexes (~2×): ~1.6 PB

**Bandwidth estimation**: peak traffic ≈ 3× average. If average is 1 Gbps, provision for 3 Gbps.

### Core Components

**Load Balancer**
- Distributes traffic across a pool of application servers
- Algorithms: round-robin, least-connections, IP-hash (session affinity), weighted
- Layer 4 (TCP) vs Layer 7 (HTTP, can route by URL/headers)
- Introduces a single point of failure → use active-active or active-passive pairs
- Hardware (F5) vs software (HAProxy, Nginx, AWS ALB)

**CDN (Content Delivery Network)**
- Caches static and cacheable content at edge nodes geographically close to users
- Reduces origin load and cross-continent latency
- Pull CDN: content fetched on first miss, cached with TTL
- Push CDN: content uploaded to CDN proactively (suitable for large files, video)
- Use for: images, JS/CSS bundles, large media, and APIs with high read:write ratio

**Cache**
- In-memory key-value store between application and database
- Redis: persistent, rich data structures (sorted sets, hashes), pub/sub, Lua scripting
- Memcached: simpler, horizontally scalable, no persistence
- Cache-aside (lazy loading): app checks cache, on miss fetches from DB and populates cache
- Write-through: writes go to cache and DB synchronously
- Write-back (write-behind): writes go to cache immediately, async flush to DB
- Eviction policies: LRU (most common), LFU, TTL-based
- Cache stampede / thundering herd: many simultaneous misses → use mutex or probabilistic early refresh

**Message Queue**
- Decouples producers from consumers; enables async processing and buffering
- Kafka: durable, distributed, replay-capable log; high throughput; consumer groups
- RabbitMQ: traditional message broker; routing, fanout, dead-letter queues
- SQS: managed, at-least-once delivery, FIFO variant for ordering
- Use cases: order processing, event sourcing, notification delivery, log aggregation

**Database: SQL vs NoSQL**

| Dimension | SQL (RDBMS) | NoSQL |
|-----------|-------------|-------|
| Schema | Fixed, normalized | Flexible / schema-on-read |
| Transactions | ACID by default | Varies (Cassandra: lightweight txns; DynamoDB: limited) |
| Query flexibility | Arbitrary JOINs, aggregations | Limited (scan is expensive) |
| Horizontal scale | Hard (sharding complex) | Designed for it |
| Consistency | Strong by default | Often eventual |
| Examples | PostgreSQL, MySQL | Cassandra, DynamoDB, MongoDB, Redis |

Choose SQL when: transactions across entities matter, schema is stable, relational queries are needed.
Choose NoSQL when: write throughput is extreme, schema is dynamic, denormalization is acceptable.

### CAP Theorem

A distributed system can guarantee at most **two** of:
- **Consistency (C)**: every read returns the most recent write
- **Availability (A)**: every request receives a (non-error) response
- **Partition Tolerance (P)**: system continues to operate despite network partitions

In practice, partitions *will* occur, so the real choice is between CP and AP:
- **CP systems**: HBase, Zookeeper, etcd — return errors under partition to preserve consistency
- **AP systems**: Cassandra, DynamoDB, CouchDB — return stale data under partition to preserve availability
- **CA systems**: only possible in single-node or non-distributed settings

See also PACELC: even when no partition exists, there is a latency vs. consistency trade-off.

### System Design Interview Framework

1. **Clarify requirements** (5 min): functional scope, scale, constraints
2. **Estimate capacity** (3 min): QPS, storage, bandwidth
3. **Define the API** (3 min): endpoints, request/response shapes
4. **High-level design** (10 min): draw boxes — clients, LB, app servers, cache, DB, queue
5. **Deep dive** (15 min): pick 2–3 components to detail; discuss trade-offs
6. **Identify bottlenecks** (5 min): SPOFs, hot partitions, cascading failures; propose mitigations

## Applications

- Designing URL shorteners (TinyURL): hash function, collision handling, redirect latency
- Designing news feeds (Twitter/Facebook): fan-out on write vs fan-out on read
- Designing rate limiters: token bucket per user, sliding window counters in Redis
- Designing search indexes: inverted index, sharding by term or document
- Designing notification systems: push vs pull, webhooks, websockets

## Trade-offs

| Choice | Pro | Con |
|--------|-----|-----|
| More caching | Low latency, reduced DB load | Stale data, memory cost, invalidation complexity |
| Async via queues | Decoupling, resilience | Eventual consistency, harder debugging |
| Microservices | Independent scaling and deployment | Network overhead, distributed tracing complexity |
| Denormalization | Faster reads | Write amplification, data inconsistency risk |
| More replicas | High availability, read scaling | Replication lag, cost |

## Links
- [[distributed_systems|Distributed Systems]]
- [[scalability_patterns|Scalability]]
