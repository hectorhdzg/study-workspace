# Distributed Systems

Deep dives into the theory and mechanisms behind distributed computing — consensus, replication, clocks, transactions, and failure handling. These topics complement the [System Design](../system-design/README.md) section, which focuses on architectural building blocks.

## Topics

- [Consensus Algorithms](consensus-algorithms.md) — Raft, Paxos, leader election, Byzantine fault tolerance
- [Replication & Partitioning](replication-partitioning.md) — Leader-follower, multi-leader, leaderless replication, consistent hashing, sharding strategies
- [Distributed Transactions](distributed-transactions.md) — 2PC, 3PC, Saga pattern, exactly-once semantics
- [Clocks & Ordering](clocks-ordering.md) — Logical clocks, vector clocks, happened-before, hybrid clocks
- [Failure Detection & Recovery](failure-detection.md) — Heartbeats, Phi accrual detector, circuit breakers, bulkheads, chaos engineering
- [Service Discovery & Coordination](service-discovery.md) — ZooKeeper, etcd, Consul, service mesh, sidecar pattern
