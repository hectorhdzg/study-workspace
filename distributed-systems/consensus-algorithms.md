# Consensus Algorithms

Consensus is the problem of getting multiple nodes to agree on a single value, even when some nodes crash or messages are delayed. It's the foundation of replicated state machines, distributed databases, and leader election.

## Why Consensus Is Hard

The **FLP impossibility result** (1985) proved that in an asynchronous system where even one process can crash, no deterministic algorithm can guarantee consensus. In practice, we work around this with timeouts, randomization, and partial synchrony assumptions.

Key challenges:
- **Network partitions** — Nodes can't tell if a peer crashed or if the network is slow.
- **Split brain** — Two nodes both think they're the leader.
- **Message reordering** — Messages can arrive out of order or be duplicated.

---

## Raft

Raft is the most widely-taught consensus algorithm, designed for understandability. It's used in etcd, CockroachDB, TiKV, and Consul.

### How It Works

Raft splits consensus into three sub-problems:

1. **Leader Election** — Nodes start as followers. If a follower doesn't hear from a leader within a random timeout, it becomes a candidate and requests votes. A candidate that receives a majority becomes the leader. The randomized timeout prevents split votes.

2. **Log Replication** — The leader accepts client requests, appends them to its log, and replicates entries to followers. An entry is **committed** once a majority of nodes have stored it. Followers apply committed entries to their state machines in order.

3. **Safety** — A candidate can only win an election if its log is at least as up-to-date as a majority of nodes. This guarantees the leader always has all committed entries.

### Raft Terms & Heartbeats

```
Term 1            Term 2            Term 3
[Leader A]  →     [Election]  →     [Leader B]
  |                  |                  |
heartbeat          timeout            heartbeat
heartbeat          vote request       heartbeat
heartbeat          vote granted       heartbeat
```

- **Term** — A logical clock that increases with each election. If a node receives a message with a higher term, it steps down.
- **Heartbeat** — The leader sends periodic empty AppendEntries RPCs to prevent election timeouts.

### Key Properties

| Property | Guarantee |
|----------|-----------|
| Election Safety | At most one leader per term |
| Leader Append-Only | Leader never overwrites or deletes log entries |
| Log Matching | If two logs have an entry with same index and term, all preceding entries are identical |
| Leader Completeness | If an entry is committed in a given term, it will be present in leaders of all higher terms |

---

## Paxos

Paxos (Lamport, 1989) is the original consensus algorithm. It's notoriously hard to understand but is the theoretical foundation for most consensus research.

### Basic Paxos (Single Decree)

Three roles (a node can play multiple roles):

1. **Proposer** — Suggests a value.
2. **Acceptor** — Votes on proposals (a quorum of acceptors must agree).
3. **Learner** — Learns the decided value.

**Phase 1 — Prepare:**
- Proposer picks a unique proposal number `n` and sends `Prepare(n)` to a majority of acceptors.
- Each acceptor promises not to accept any proposal numbered less than `n`. If it has already accepted a proposal, it returns the highest-numbered one.

**Phase 2 — Accept:**
- If the proposer receives promises from a majority, it sends `Accept(n, value)` where value is either the highest-numbered previously accepted value, or its own value if no acceptor reported one.
- Acceptors accept the proposal unless they've already promised a higher number.

### Multi-Paxos

Basic Paxos decides a single value. Multi-Paxos extends it to a sequence of values (a replicated log) by electing a stable leader who skips Phase 1 for subsequent proposals. This is what's actually used in practice — Raft is essentially a more structured Multi-Paxos.

---

## Leader Election

Many distributed systems need a single coordinator (leader) for simplicity. Common approaches:

| Approach | How It Works | Used By |
|----------|-------------|---------|
| Raft-based | Consensus on leader identity; majority vote | etcd, Consul |
| Bully Algorithm | Highest-ID node wins; nodes challenge lower-ID leaders | Simple systems |
| Lease-based | Leader holds a time-limited lease; must renew before expiry | Chubby, ZooKeeper |
| Fencing tokens | Each leader epoch gets a monotonically increasing token; stale leaders are rejected | Distributed locks |

### Fencing Tokens

A critical pattern: when a leader acquires a lock/lease, it also gets a **fencing token** (an incrementing number). All downstream operations include this token. If a stale leader (with a lower token) tries to write, the storage layer rejects it.

```
Leader A gets lock → token 33
Leader A pauses (GC, network)
Leader B gets lock → token 34
Leader B writes with token 34 ✓
Leader A wakes up, writes with token 33 ✗ (rejected: 33 < 34)
```

---

## Byzantine Fault Tolerance (BFT)

Standard consensus (Raft, Paxos) assumes nodes are **crash-fault** — they either work correctly or stop. **Byzantine faults** cover nodes that behave arbitrarily (bugs, corruption, malicious actors).

- **PBFT (Practical BFT)** — Tolerates up to f Byzantine nodes with 3f + 1 total nodes. Uses a three-phase protocol (pre-prepare, prepare, commit). Used in some blockchain systems.
- **When you need it** — Financial systems, blockchain, multi-organization systems where you can't trust all participants.
- **When you don't** — Internal datacenter systems where you control all nodes. Crash-fault tolerance (Raft) is simpler and sufficient.

---

## Interview Tips

- Know Raft well enough to explain leader election and log replication on a whiteboard.
- Understand **why** consensus is needed: replicated state machines, distributed locks, configuration management.
- Be able to discuss trade-offs: Raft is easier to implement; Paxos is more flexible; BFT is expensive.
- Common question: "What happens during a network partition?" — The partition with a majority can still make progress; the minority side is blocked.

---

## Practice Questions

- How does Raft handle a network partition that splits the cluster in half?
- What's the difference between Raft and Paxos in practice?
- Why do we need fencing tokens even with a consensus-based lock?
- When would you choose BFT over crash-fault tolerance?
