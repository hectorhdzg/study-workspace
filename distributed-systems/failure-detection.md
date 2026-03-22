# Failure Detection & Recovery

In a distributed system, failures are the norm, not the exception. Nodes crash, networks partition, disks corrupt, and entire datacenters go offline. The system must detect failures quickly, contain their blast radius, and recover gracefully.

---

## Types of Failures

| Failure Type | Description | Example |
|-------------|-------------|---------|
| Crash failure | Node stops and doesn't recover | Server hardware failure |
| Omission failure | Node fails to send or receive messages | Dropped packets, full queue |
| Timing failure | Response arrives outside expected time bounds | GC pause, overloaded node |
| Byzantine failure | Node behaves arbitrarily (including maliciously) | Bug, corruption, compromised node |
| Network partition | Communication between groups of nodes is lost | Datacenter link failure |

---

## Failure Detection

### Heartbeat-Based Detection

The simplest approach: nodes periodically send heartbeat messages to a monitor (or to each other). If a heartbeat is missed for a configurable timeout, the node is declared failed.

**The tension:** Short timeouts detect failures quickly but produce false positives (a slow node or brief network blip looks like a crash). Long timeouts reduce false positives but delay failure detection.

### Phi Accrual Failure Detector

Instead of a binary "alive/dead" decision, the Phi accrual detector outputs a **suspicion level** (φ) that increases as heartbeats are delayed.

- Tracks the arrival time of heartbeats and builds a statistical model of the inter-arrival distribution.
- φ represents the likelihood (on a log scale) that the node has failed.
- The application chooses a threshold — higher thresholds mean fewer false positives but slower detection.

**Used by:** Apache Cassandra, Akka

### Gossip-Based Detection

Each node periodically picks a random peer and exchanges its view of cluster membership (who's alive, who's suspected dead). Failures propagate through the cluster logarithmically — after a few gossip rounds, all nodes have a consistent view.

**Advantages:** Decentralized (no single monitor), tolerates network partitions gracefully, scales well.

**Used by:** Cassandra, Consul, SWIM protocol

---

## Fault Tolerance Patterns

### Circuit Breaker

Prevents a service from repeatedly calling a failing dependency. The circuit breaker sits between the caller and the dependency, tracking failures.

**States:**
- **Closed** (normal) — Requests pass through. If failures exceed a threshold, trip to Open.
- **Open** — All requests fail immediately without calling the dependency. After a timeout, move to Half-Open.
- **Half-Open** — Allow a few test requests through. If they succeed, return to Closed. If they fail, back to Open.

```
         success
Closed ←─────────── Half-Open
  │                    ↑
  │ failure threshold  │ timeout
  ↓                    │
  Open ────────────────┘
```

**Why it matters:** Without a circuit breaker, a failing service can cause cascading failures — the caller's thread pool fills up waiting for timeouts, then the caller also becomes unresponsive.

### Bulkhead

Isolate different parts of the system so a failure in one doesn't exhaust shared resources and bring down everything else. Named after ship bulkheads that contain flooding.

**Implementation approaches:**
- **Thread pool isolation** — Each dependency gets its own thread pool. If Service A's pool fills up, requests to Service B are unaffected.
- **Connection pool isolation** — Separate connection pools per dependency.
- **Process isolation** — Run different workloads in separate processes or containers.

### Retry with Backoff

When a transient failure occurs (network blip, temporary overload), retrying can succeed. But naive retries can make things worse — a failing service gets hit with even more requests.

**Best practices:**
- **Exponential backoff** — Wait 1s, 2s, 4s, 8s, etc. between retries.
- **Jitter** — Add randomness to the delay so retried requests don't arrive in a synchronized burst.
- **Max retries** — Cap the total number of attempts.
- **Idempotency** — Only retry if the operation is safe to repeat (or use idempotency keys).

### Timeout Budgets

Give each request a total time budget that shrinks as it passes through services. If Service A calls Service B which calls Service C, and the total budget is 500ms, Service B should pass a reduced budget (e.g., 300ms) to Service C so there's time to handle the response.

---

## Recovery Patterns

### Write-Ahead Log (WAL)

Before modifying data, write the intended change to a durable log. If the node crashes mid-operation, it can replay the log on restart to recover to a consistent state.

**Used by:** Every major database (PostgreSQL, MySQL, SQLite), Kafka, etcd

### Checkpointing

Periodically save a snapshot of the current state. On recovery, load the latest checkpoint and replay only the logs written after it — much faster than replaying the entire history.

### Read Repair & Anti-Entropy

In eventually consistent systems, replicas may diverge:
- **Read repair** — When a read detects stale data on some replicas, immediately update them.
- **Anti-entropy (Merkle trees)** — Background process compares data across replicas using hash trees, efficiently identifying and syncing differences.

---

## Chaos Engineering

Proactively inject failures into production to test resilience:

| Tool | What It Does | Created By |
|------|-------------|-----------|
| Chaos Monkey | Randomly kills VMs in production | Netflix |
| Chaos Kong | Simulates entire region failure | Netflix |
| Litmus | Kubernetes-native chaos experiments | CNCF |
| Gremlin | Managed chaos-as-a-service | Gremlin Inc. |

**Principles:**
1. Define a "steady state" (normal behavior metrics).
2. Hypothesize that steady state continues during the experiment.
3. Inject real-world failures (node crash, network latency, disk full).
4. Look for differences between control and experimental groups.
5. Minimize blast radius — start small.

---

## Interview Tips

- Circuit breaker + retry with backoff is the standard answer for "how do you handle dependent service failures?"
- Know the difference between fail-fast (circuit breaker open) and fail-safe (fallback/degraded response).
- Be able to discuss cascading failures and how to prevent them (bulkheads, circuit breakers, load shedding).
- Chaos engineering shows maturity — mention it when discussing how you'd validate a system's fault tolerance.

---

## Practice Questions

- A microservice depends on 5 downstream services. One becomes slow (not failing, just slow). How does this affect the whole system, and what patterns prevent cascading failure?
- Design a failure detection system for a 1000-node cluster. Why might gossip-based detection be better than centralized heartbeats?
- Explain why exponential backoff with jitter is critical for retry strategies.
- How would you test that your system handles a full AWS region failure?
