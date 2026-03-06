---
layer: 03_software_engineering
type: engineering
tool: NoSQL
status: growing
tags: [pattern, retrieval]
created: 2026-03-05
---

# NoSQL Patterns

## Purpose

Reference for NoSQL data models, access patterns, and selection criteria across the four
major NoSQL categories (document, key-value, wide-column, graph) plus vector databases.
Knowing which store fits a workload is as important as knowing how to use each one.

## Architecture

### NoSQL Category Overview

| Category | Representative DBs | Data model | Query strength |
|---|---|---|---|
| Document | MongoDB, Firestore, CouchDB | JSON-like nested docs | Rich queries on nested fields |
| Key-value | Redis, DynamoDB, etcd | Opaque values by key | O(1) point lookups |
| Wide-column | Cassandra, HBase, Bigtable | Rows with dynamic column families | High-throughput time-series, partitioned scans |
| Graph | Neo4j, Amazon Neptune, ArangoDB | Nodes + edges with properties | Multi-hop traversals, relationship queries |
| Vector | pgvector, Qdrant, Weaviate, Pinecone | Embedding vectors + metadata | Approximate nearest neighbour (ANN) search |

**Rule of thumb**: let the access pattern choose the store, not the other way around.

---

### Document Databases (MongoDB)

Schema design: **embedding vs referencing**

```js
// EMBEDDING — one document holds the whole aggregate
// Good: read the whole thing in one query; data is owned by parent
{
  "_id": ObjectId("..."),
  "title": "Order #1042",
  "customer": { "name": "Ada", "email": "ada@example.com" },
  "items": [
    { "sku": "A1", "qty": 2, "price": 9.99 },
    { "sku": "B3", "qty": 1, "price": 24.99 }
  ]
}

// REFERENCING — store foreign key, join at application layer
// Good: large or independently updated sub-documents
{
  "_id": ObjectId("..."),
  "title": "Order #1042",
  "customer_id": ObjectId("cust-456"),  // separate customers collection
  "item_ids": [ObjectId("item-1"), ObjectId("item-2")]
}
```

**Embedding guidelines:**
- Embed when child data is always accessed with the parent.
- Embed when child is not shared across multiple parents.
- Avoid embedding unbounded arrays (document size limit: 16 MB in MongoDB).

**Indexing in MongoDB:**
```js
// Single field
db.orders.createIndex({ customer_id: 1 });

// Compound (order matters — ESR rule: Equality, Sort, Range)
db.orders.createIndex({ status: 1, created_at: -1, total: 1 });

// Text search
db.products.createIndex({ name: "text", description: "text" });

// TTL index: auto-expire documents
db.sessions.createIndex({ expires_at: 1 }, { expireAfterSeconds: 0 });

// Partial index
db.orders.createIndex(
  { created_at: -1 },
  { partialFilterExpression: { status: "pending" } }
);
```

**Aggregation pipeline:**
```js
db.orders.aggregate([
  { $match: { status: "completed" } },
  { $group: { _id: "$customer_id", total: { $sum: "$amount" } } },
  { $sort: { total: -1 } },
  { $limit: 10 },
  { $lookup: {
      from: "customers",
      localField: "_id",
      foreignField: "_id",
      as: "customer"
  }},
  { $unwind: "$customer" },
  { $project: { "customer.name": 1, total: 1 } }
]);
```

---

### Key-Value: Redis

Redis data structures and use cases:

```python
import redis
r = redis.Redis()

# String — simple cache, counters, rate limits
r.set("user:42:name", "Ada", ex=3600)          # TTL in seconds
r.incr("rate:ip:192.0.2.1")                    # atomic increment
r.expire("rate:ip:192.0.2.1", 60)

# Hash — object fields without JSON (de)serialization
r.hset("user:42", mapping={"name": "Ada", "email": "ada@x.com"})
r.hgetall("user:42")

# List — queues, activity feeds (FIFO or LIFO)
r.rpush("job_queue", "job:100", "job:101")
r.blpop("job_queue", timeout=5)               # blocking pop (worker pattern)

# Set — unique membership, tags, deduplication
r.sadd("tags:post:99", "python", "ml")
r.sismember("tags:post:99", "python")         # O(1)
r.sinter("tags:post:99", "tags:post:100")     # common tags

# Sorted Set — leaderboards, rate limiters, priority queues
r.zadd("leaderboard", {"alice": 9800, "bob": 9200})
r.zrevrange("leaderboard", 0, 9, withscores=True)  # top 10

# Stream — append-only event log (Redis Streams, Kafka-lite)
r.xadd("events", {"type": "click", "user": "42"})
r.xread(count=100, streams={"events": "0"})
```

**Redis as Pub/Sub:**
```python
# Publisher
r.publish("notifications", '{"user":42,"msg":"hello"}')

# Subscriber (blocking)
p = r.pubsub()
p.subscribe("notifications")
for msg in p.listen():
    if msg["type"] == "message":
        handle(msg["data"])
```

---

### Wide-Column: Cassandra

Cassandra is designed for **write-heavy, high-availability, partitioned** workloads.
Schema design is **query-first**: define tables to match specific read patterns.

```cql
-- Partition key determines which node holds the data
-- Clustering columns determine on-disk sort order within a partition
CREATE TABLE sensor_readings (
    sensor_id  UUID,
    recorded_at TIMESTAMP,
    value      DOUBLE,
    PRIMARY KEY (sensor_id, recorded_at)
) WITH CLUSTERING ORDER BY (recorded_at DESC);

-- Query MUST include partition key; range on clustering columns is fine
SELECT * FROM sensor_readings
WHERE sensor_id = ? AND recorded_at > '2026-01-01'
LIMIT 1000;

-- Denormalize for different access patterns (duplicate data is expected)
CREATE TABLE readings_by_region (
    region     TEXT,
    recorded_at TIMESTAMP,
    sensor_id  UUID,
    value      DOUBLE,
    PRIMARY KEY (region, recorded_at, sensor_id)
) WITH CLUSTERING ORDER BY (recorded_at DESC);
```

**Cassandra anti-patterns:**
- Never do `SELECT *` without a partition key filter (full-cluster scan).
- Avoid large partitions (>100 MB) — use time-bucketing (e.g., partition by `(sensor_id, month)`).
- Avoid `ALLOW FILTERING` in production — it signals a missing index or wrong schema.

---

### Graph Databases (Neo4j / Cypher)

```cypher
// Create nodes and relationships
CREATE (a:Person {name: "Ada"})
CREATE (b:Person {name: "Bob"})
CREATE (a)-[:FOLLOWS]->(b)

// Variable-length path: friends of friends up to depth 3
MATCH (ada:Person {name: "Ada"})-[:FOLLOWS*1..3]->(other)
RETURN other.name

// Shortest path
MATCH p = shortestPath(
  (ada:Person {name:"Ada"})-[:FOLLOWS*]-(bob:Person {name:"Bob"})
)
RETURN length(p), p

// Recommendation: people Ada doesn't follow yet but her followees do
MATCH (ada:Person {name:"Ada"})-[:FOLLOWS]->(friend)-[:FOLLOWS]->(rec)
WHERE NOT (ada)-[:FOLLOWS]->(rec) AND ada <> rec
RETURN rec.name, COUNT(friend) AS mutual
ORDER BY mutual DESC
LIMIT 10
```

**When graph DBs win:**
- Multi-hop relationship queries (social graphs, fraud rings, recommendation).
- Schema-flexible relationships with arbitrary properties.
- Graph algorithms: PageRank, community detection, betweenness centrality.

---

### Vector Databases

Vector DBs store dense float vectors (embeddings) and answer **approximate nearest
neighbour (ANN)** queries: "find the K most similar vectors to this query vector."

**Core concepts:**

- **Embedding**: model output representing semantic content as a vector (e.g., 1536-dim
  for `text-embedding-ada-002`).
- **HNSW index** (Hierarchical Navigable Small World): layered graph index, O(log n)
  insert and query; gold standard for ANN accuracy/speed.
- **IVF** (Inverted File Index): clusters vectors; faster to build, slightly lower recall.
- **ANN search**: trades exact recall for speed. Tune `ef_search` (HNSW) or `nprobe`
  (IVF) to balance latency vs recall.

**pgvector (PostgreSQL extension):**
```sql
CREATE EXTENSION vector;
CREATE TABLE documents (
    id     BIGSERIAL PRIMARY KEY,
    text   TEXT,
    embedding VECTOR(1536)
);

-- HNSW index
CREATE INDEX ON documents USING hnsw (embedding vector_cosine_ops)
WITH (m = 16, ef_construction = 64);

-- ANN query with metadata filter
SELECT id, text, 1 - (embedding <=> $1::vector) AS similarity
FROM documents
WHERE category = 'research'
ORDER BY embedding <=> $1::vector
LIMIT 10;
```

**Qdrant (standalone vector DB):**
```python
from qdrant_client import QdrantClient
from qdrant_client.models import VectorParams, Distance, PointStruct, Filter, FieldCondition, MatchValue

client = QdrantClient("localhost", port=6333)

client.create_collection(
    collection_name="docs",
    vectors_config=VectorParams(size=1536, distance=Distance.COSINE),
)

client.upsert("docs", points=[
    PointStruct(id=1, vector=embed("hello world"), payload={"category": "intro"})
])

results = client.search(
    collection_name="docs",
    query_vector=embed("greeting"),
    query_filter=Filter(must=[FieldCondition(key="category", match=MatchValue(value="intro"))]),
    limit=5,
)
```

**Filtering considerations:**
- Pre-filtering (filter then search): safe but can reduce recall if filtered set is tiny.
- Post-filtering (search then filter): may return fewer than K results.
- Qdrant uses segment-level filtering to maintain recall during pre-filter.

## Implementation Notes

### Choosing a Store

```
Is data relational with complex queries? → PostgreSQL
Need sub-millisecond cache / session store? → Redis
High-write time-series or IoT data, multi-region? → Cassandra
Relationship-centric queries (hops, paths)? → Neo4j
Semantic / similarity search over embeddings? → pgvector or Qdrant
Documents with flexible schema, rich queries? → MongoDB
```

### Schema Migration Strategies

- **Document DBs**: schema-on-read means migrations are code changes, not DDL.
  Use version fields (`schema_version: 2`) and migrate lazily on read.
- **Cassandra**: additive-only DDL is safe. Dropping columns is risky — tombstones.
- **Vector DBs**: re-embedding is required when changing the embedding model.
  Maintain two collections during migration; gradually shift traffic.

## Trade-offs

| Store | Strength | Weakness |
|---|---|---|
| MongoDB | Rich queries, flexible schema | No ACID across collections by default (txns added in 4.0) |
| Redis | Extreme speed, rich structures | Data fits in RAM; persistence is eventual by default |
| Cassandra | Linear horizontal scale, write throughput | No ad-hoc queries; schema must match access patterns |
| Neo4j | Relationship traversals | Poor for large analytical graph workloads; single-master |
| pgvector | Stays in PostgreSQL, ACID | Slower than dedicated vector DBs at scale (>10M vectors) |
| Qdrant | Best-in-class ANN + filtering | Separate system to operate |

## References

- [MongoDB Schema Design Patterns](https://www.mongodb.com/blog/post/building-with-patterns-a-summary)
- [Cassandra Data Modeling Guide](https://cassandra.apache.org/doc/latest/cassandra/data_modeling/)
- [pgvector README](https://github.com/pgvector/pgvector)
- [Qdrant documentation](https://qdrant.tech/documentation/)
- [Neo4j Cypher Reference](https://neo4j.com/docs/cypher-manual/current/)
- *Designing Data-Intensive Applications* — Martin Kleppmann (Ch. 2, 3)

## Links
- [[sql_patterns|SQL Patterns]]
- [[caching_strategies|Caching Strategies]]
- [[distributed_systems|Distributed Systems]]
- [[05_ai_engineering/03_rag_and_agents/vector_stores|Vector Stores]]
