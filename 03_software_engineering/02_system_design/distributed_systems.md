---
layer: 03_software_engineering
type: concept
status: growing
tags: [distributed-systems, consensus, consistency, fault-tolerance, patterns]
created: 2026-03-05
---

# Distributed Systems

## Definition

A distributed system is a collection of autonomous computing nodes that communicate over a network and coordinate to appear as a single coherent system to end users. The defining properties are: nodes are physically separated, communication is unreliable (messages can be lost, delayed, or reordered), and nodes can fail independently.

## Intuition

The core challenge of distributed systems is that you cannot have simultaneity: there is no shared clock, no instantaneous communication, no global observer. Every assertion like "node A and node B agree on X" must be achieved through a protocol that tolerates message delays and failures. Most distributed systems problems reduce to: how do we make nodes agree on something (consensus) while tolerating failures?

## Formal Description

### Consistency Models

Consistency models define what guarantees a distributed database gives about the order in which operations are observed across nodes.

**Strong consistency (linearizability)**
- Every operation appears to take effect instantaneously at some point between its invocation and completion
- All clients observe the same total order of operations
- Equivalent to a single-machine system from the client's perspective
- Cost: high latency (requires coordination on every write), reduced availability under partitions
- Examples: etcd, Zookeeper, Google Spanner (with TrueTime)

**Sequential consistency**
- All operations appear in *some* sequential order that is consistent with the order seen by each individual process
- Weaker than linearizability: the chosen order need not align with real-time
- Used in some CPU memory models

**Causal consistency**
- Operations that are causally related (A happened-before B) are seen in that order by all nodes
- Concurrent operations (no causal relation) may be seen in different orders by different nodes
- Stronger than eventual, weaker than strong; often a practical sweet spot
- Examples: MongoDB causal sessions, some CRDTs

**Eventual consistency**
- If no new updates are made, all replicas will eventually converge to the same value
- No bound on how long convergence takes
- High availability and low latency at the cost of temporary divergence
- Requires conflict resolution: last-write-wins (LWW), CRDTs, application-level merging
- Examples: Cassandra, DynamoDB (default), S3

**Read-your-writes consistency**
- A client always reads its own most recent write (even if other clients may not yet see it)
- Implemented by routing a client's reads to the replica that received its writes, or by version vectors

**Monotonic reads**
- Once a client reads a value, subsequent reads will never return an older value
- Prevents a confusing "time traveling" effect where a page load shows older data than a previous one

### Consensus: Paxos and Raft

Consensus protocols allow a cluster of nodes to agree on a single value (or sequence of values) even in the presence of failures, as long as a quorum (majority) of nodes is reachable.

**Paxos (Lamport, 1989)**
- Roles: Proposer, Acceptor, Learner
- Two phases: Prepare/Promise (elect a leader), Accept/Accepted (commit a value)
- Single-decree Paxos agrees on one value; Multi-Paxos extends to a log by reusing the leader
- Notoriously hard to implement correctly; leaves many details underspecified

**Raft (Ongaro & Ousterhout, 2014)**
- Designed for understandability; equivalent safety guarantees to Multi-Paxos
- Leader-based: one leader per term, elected by majority vote
- Log replication: leader appends entries, replicates to followers, commits once majority acknowledges
- Key invariant: a node cannot become leader unless its log is at least as up-to-date as the majority
- Leader election uses randomized timeouts to avoid split votes
- Used in: etcd, CockroachDB, TiKV, Consul

**Quorum writes/reads**
- With N replicas, a write quorum W and read quorum R, requiring W + R > N guarantees read-your-writes
- Example: N=3, W=2, R=2. Any pair of read and write sets must overlap by at least one replica
- Tunable in Cassandra: ALL, QUORUM, LOCAL_QUORUM, ONE

### Leader Election

Beyond Raft, simpler leader election can use:
- **Bully algorithm**: highest-ID node wins; O(N²) messages
- **Ring algorithm**: token passed around ring; O(N) messages
- **Zookeeper ephemeral nodes**: first to create `/leader` wins; watchers for failure detection
- **Distributed locks** (Redlock on Redis): controversial; Martin Kleppmann's critique argues it cannot guarantee safety under clock skew

### Failure Modes

**Partial failure** — the defining characteristic of distributed systems. Node A may fail while B continues. Unlike a single machine where a crash is total, partial failures create ambiguity: did the remote call succeed or fail? The receiving node might have processed the request before crashing.

**Split-brain** — a network partition causes two sub-clusters each believing they are the sole authoritative partition. Without a quorum requirement, both halves may accept writes, leading to divergent state. Solution: require a majority quorum for writes (sacrificing availability in the minority partition).

**Network partitions** — network link failure separates nodes. Messages may be dropped, delayed indefinitely, or reordered. TCP guarantees ordering and delivery *within a connection* but not across connections or after reconnect.

**Byzantine failures** — nodes behave arbitrarily maliciously or send contradictory messages to different nodes. Requires Byzantine Fault Tolerant (BFT) protocols (PBFT, Tendermint). Most datacenter systems assume crash-stop failures only.

**Clock skew** — physical clocks on different machines drift. NTP reduces but does not eliminate drift (typically ±100 ms). Logical clocks (Lamport timestamps, vector clocks) capture causality without relying on wall clocks.

**Cascading failures** — a struggling node responds slowly, causing upstream timeouts and retries, increasing load further. Circuit breakers, bulkheads, and load shedding break the cascade.

### Patterns

**Saga pattern**
- Manages distributed transactions across services without two-phase commit
- A saga is a sequence of local transactions; each step publishes an event or message
- On failure, compensating transactions undo the preceding steps
- Choreography: services react to events (decentralized, harder to monitor)
- Orchestration: a central saga orchestrator issues commands and handles failures
- Example: booking a flight + hotel + car — if car fails, cancel flight and hotel

**Circuit Breaker**
- Wraps remote calls; tracks failure rate over a sliding window
- States: Closed (normal) → Open (failing fast, no calls attempted) → Half-Open (probe)
- Prevents wasted resources on calls to a down service and gives the service time to recover
- Hystrix (Netflix), Resilience4j, Polly (.NET)

**Bulkhead**
- Isolates thread pools (or connection pools) per downstream service
- A slow or failing service exhausts only its own pool, not the shared pool
- Analogy: watertight compartments on a ship; one flooded compartment does not sink the ship

**Retry with exponential backoff and jitter**
- On transient failure, retry after 2^attempt × base_delay + random_jitter
- Jitter prevents synchronized retry storms (all clients retrying simultaneously)
- Idempotency keys are essential: safe to retry only if the operation is idempotent
- Distinguish retryable errors (503, network timeout) from non-retryable (400, 401)

**Two-Phase Commit (2PC)**
- Coordinator sends Prepare to all participants; waits for all votes
- If all vote Yes, coordinator sends Commit; otherwise Abort
- Blocking protocol: if coordinator crashes after Prepare, participants are locked until it recovers
- Use distributed sagas or XA transactions only where truly necessary

### CAP and PACELC

**CAP theorem** (Brewer 2000, proved by Gilbert & Lynch 2002): under a network Partition, choose between Consistency and Availability.

**PACELC** (Abadi 2012): extends CAP by noting that even when there is no partition (the else case), there is always a trade-off between Latency and Consistency:
- DynamoDB/Cassandra: PA/EL (sacrifice consistency for availability and low latency)
- etcd/Zookeeper: PC/EC (sacrifice availability and accept higher latency for consistency)
- Google Spanner: PC/EL (strong consistency even at high latency, using TrueTime)

## Applications

- Distributed databases and key-value stores (Cassandra, DynamoDB, CockroachDB)
- Distributed coordination services (Zookeeper, etcd)
- Distributed messaging systems (Kafka — partition leaders, ISR, offset commits)
- Microservice architectures with saga-based transaction management
- Distributed caches (Redis Cluster with hash slots)
- Stream processing systems (Flink, Kafka Streams — state management and checkpointing)

## Trade-offs

| Approach | Pro | Con |
|----------|-----|-----|
| Strong consistency | Simple application logic | Higher latency, lower availability |
| Eventual consistency | High availability, low latency | Complex conflict resolution, surprising behavior |
| Saga over 2PC | No distributed lock, resilient | Compensating transactions are complex to write |
| Leader-based replication | Simple reasoning | Leader is a bottleneck and SPOF risk |
| Leaderless (Dynamo-style) | No single bottleneck | Conflict resolution required (vector clocks, LWW) |
| Synchronous replication | Durability guaranteed | Write latency = slowest replica |
| Asynchronous replication | Low write latency | Risk of data loss if primary fails before replica catches up |

## Links
- [[system_design_fundamentals|System Design]]
- [[scalability_patterns|Scalability Patterns]]
- [[caching_strategies|Caching Strategies]]
- [[nosql_patterns|NoSQL Patterns]]
- [[04_ml_engineering/01_data_engineering/data_pipeline_patterns|Data Pipeline Patterns]]
