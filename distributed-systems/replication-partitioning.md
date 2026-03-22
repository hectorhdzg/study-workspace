# Replication & Partitioning

Replication (copying data to multiple nodes) and partitioning (splitting data across nodes) are the two fundamental strategies for scaling distributed data systems. They're often used together.

---

## Why Replicate?

- **Availability** — If one node fails, others can serve traffic.
- **Read throughput** — Distribute read load across replicas.
- **Latency** — Place replicas closer to users geographically.

The core trade-off: more replicas improve availability and read performance, but make writes harder to keep consistent.

---

## Replication Strategies

### Leader-Follower (Primary-Secondary)

One node (the leader) handles all writes and replicates changes to followers. Followers serve reads.

```
Client → [Leader] ──write──→ [Follower 1]
                   ──write──→ [Follower 2]
         [Leader] ←──read───  Client
                              [Follower 1] ←──read── Client
```

**Synchronous vs Asynchronous replication:**

| Mode | Behavior | Trade-off |
|------|----------|-----------|
| Synchronous | Leader waits for follower ACK before confirming write | Strong consistency, but higher latency and lower availability |
| Asynchronous | Leader confirms immediately, replicates in background | Fast writes, but followers may lag (stale reads) |
| Semi-synchronous | One follower is sync, rest are async | Guarantees at least two copies before confirming |

**Failover** — When the leader fails, a follower must be promoted. This is tricky:
- How do you detect the leader is down? (Timeout-based, risk of false positives)
- Which follower should be promoted? (Most up-to-date one)
- What happens to writes the old leader accepted but didn't replicate? (Data loss risk with async replication)

**Used by:** PostgreSQL, MySQL, MongoDB (replica sets), Redis Sentinel

### Multi-Leader (Multi-Primary)

Multiple nodes accept writes, then replicate to each other. Useful for multi-datacenter setups where each datacenter has its own leader.

**The problem: write conflicts.** Two leaders can concurrently modify the same record.

Conflict resolution strategies:
- **Last-write-wins (LWW)** — Use timestamps; latest write wins. Simple but loses data.
- **Merge** — Application-specific logic to combine conflicting writes (e.g., CRDTs for automatic conflict-free merging).
- **Custom resolution** — Push conflicts to the application layer (e.g., show both versions to the user like Git).

**Used by:** CouchDB, Dynamo-style systems, multi-datacenter PostgreSQL (BDR)

### Leaderless (Dynamo-Style)

Any node can accept reads and writes. The client sends requests to multiple nodes in parallel and uses quorums to determine success.

**Quorum condition:** For `n` replicas, `w` write nodes, and `r` read nodes:
- **w + r > n** guarantees at least one node in a read has the latest write.
- Common: n=3, w=2, r=2 (can tolerate 1 node failure for both reads and writes).

**Read repair** — When a read detects stale data on some replicas, it updates them in the background.

**Anti-entropy** — A background process continuously compares replicas and fixes inconsistencies.

**Used by:** Amazon Dynamo, Apache Cassandra, Riak, Voldemort

---

## Partitioning (Sharding)

Partitioning splits data across nodes so no single node must store everything. Each partition (shard) holds a subset of the data.

### Partitioning Strategies

| Strategy | How It Works | Pros | Cons |
|----------|-------------|------|------|
| Range-based | Assign contiguous key ranges to partitions (e.g., A-M, N-Z) | Range queries are efficient | Hot spots if keys aren't uniformly distributed |
| Hash-based | Hash the key, assign to partition based on hash | Even distribution | Range queries require scatter-gather across all partitions |
| Directory-based | A lookup table maps keys to partitions | Flexible | The directory is a single point of failure |

### Consistent Hashing

Standard hash-based partitioning breaks when you add or remove nodes (all keys would need to be remapped). **Consistent hashing** minimizes remapping by arranging nodes on a virtual ring:

1. Hash both nodes and keys onto a ring (0 to 2^32).
2. Each key is assigned to the next node clockwise on the ring.
3. When a node is added/removed, only keys between the new/removed node and its predecessor are affected — roughly `1/n` of all keys.

**Virtual nodes** — Each physical node gets multiple positions on the ring for better balance. A node with more capacity gets more virtual nodes.

```
         Node A (v1)
        /           \
Key X →              Node B (v1)
        \           /
         Node C (v1)
        /           \
Node A (v2)          Node B (v2)
```

**Used by:** Amazon Dynamo, Apache Cassandra, Memcached, CDN routing

### Rebalancing

When nodes join or leave, data must be redistributed:
- **Fixed number of partitions** — Create many more partitions than nodes; assign partitions to nodes. Rebalancing moves whole partitions. (Used by Riak, Elasticsearch, Couchbase)
- **Dynamic partitioning** — Split partitions when they get too large, merge when too small. (Used by HBase, RethinkDB)
- **Proportional to nodes** — Each node gets a fixed number of partitions; when a new node joins, it steals random partitions from existing nodes. (Used by Cassandra)

---

## Secondary Indexes in Partitioned Systems

Secondary indexes are tricky with partitioned data:

- **Local indexes (document-partitioned)** — Each partition maintains its own index for the data it holds. Writes are fast (update one partition), but reads must scatter-gather across all partitions.
- **Global indexes (term-partitioned)** — The index itself is partitioned across nodes. Reads hit one partition, but writes may need to update multiple index partitions.

---

## Interview Tips

- Know the leader-follower model and its failure modes (split brain, replication lag).
- Be able to explain consistent hashing with virtual nodes on a whiteboard.
- Understand quorum math: why w + r > n ensures consistency, and what happens when it doesn't hold.
- Common question: "How would you handle a hot partition?" — Sub-partition the hot key, add a random suffix to spread load, or use a dedicated cache.

---

## Practice Questions

- Compare leader-follower vs leaderless replication for a global e-commerce platform.
- How does consistent hashing handle node failure? What about uneven load?
- What happens in a Dynamo-style system if a write succeeds on w nodes but one of those nodes crashes before replicating?
- Design a sharding strategy for a social media platform where some users have millions of followers.
