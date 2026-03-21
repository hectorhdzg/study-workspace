# Core System Design Concepts

## CAP Theorem

Any distributed system can guarantee at most **2 of 3**:

| Property            | Description                                              |
|--------------------|----------------------------------------------------------|
| **Consistency**     | Every read receives the most recent write                |
| **Availability**    | Every request receives a response (not necessarily fresh)|
| **Partition Tolerance** | System continues despite network splits            |

Since network partitions are unavoidable in distributed systems, you must choose between **CP** or **AP**.

- **CP systems:** HBase, ZooKeeper — consistent but may be unavailable during partition.
- **AP systems:** DynamoDB, Cassandra, CouchDB — always available but may return stale data.

---

## Consistency Models

| Model              | Description                                              |
|-------------------|----------------------------------------------------------|
| **Strong**         | All reads reflect the latest write (linearizable)        |
| **Eventual**       | Reads may be stale, but eventually consistent            |
| **Read-your-writes**| You always see your own writes                          |
| **Causal**         | Causally related operations seen in order                |

---

## ACID vs BASE

| ACID (Relational DBs)          | BASE (NoSQL)                               |
|-------------------------------|---------------------------------------------|
| Atomicity                     | Basically Available                         |
| Consistency                   | Soft state                                  |
| Isolation                     | Eventual consistency                        |
| Durability                    |                                             |

---

## Latency Numbers Every Engineer Should Know

| Operation                          | Approximate Latency |
|-----------------------------------|---------------------|
| L1 cache reference                | 0.5 ns              |
| L2 cache reference                | 7 ns                |
| Main memory (RAM) reference       | 100 ns              |
| Read 1 MB sequentially from RAM   | 250 µs              |
| SSD random read                   | 150 µs              |
| Read 1 MB from SSD                | 1 ms                |
| Disk seek                         | 10 ms               |
| Round trip in same data center    | 500 µs              |
| Round trip California → Netherlands| 150 ms             |

**Key takeaways:**
- Memory is ~1000x faster than disk.
- Network round trips add significant latency — minimize them.
- CDNs serve assets close to users to reduce latency.

---

## Estimation Formulas

```
Queries per second = Daily Active Users × Requests per user per day / 86,400
Storage per year   = Writes per day × Payload size × 365
Bandwidth          = Requests per second × Payload size
```

### Example: Twitter-Scale Estimation

- 300M daily active users, 5 tweets/day each = 1,500M tweets/day ≈ **17,400 tweets/second**
- Each tweet = 300 bytes → 450 GB/day storage
- Read:Write ratio = 100:1 → 1.74M reads/second

---

## Horizontal vs Vertical Scaling

| Dimension           | Vertical (Scale Up)        | Horizontal (Scale Out)    |
|--------------------|----------------------------|---------------------------|
| Approach            | Bigger, more powerful server| More servers               |
| Limit               | Hardware limit             | Near-infinite              |
| Cost                | Expensive at high end      | Commodity hardware        |
| Failure tolerance   | Single point of failure    | More resilient             |
| Complexity          | Simple                     | Requires load balancing   |

---

## Stateless vs Stateful Architecture

- **Stateless servers** — No session state stored on server. Easy to scale horizontally. State lives in DB or client.
- **Stateful servers** — Session state stored in server memory. Sticky sessions or session replication required.

**Prefer stateless** for scalable web services.
