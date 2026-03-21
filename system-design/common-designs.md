# Common System Designs

## 1. URL Shortener (e.g., bit.ly)

### Requirements
- Shorten a URL and return a 6-character code.
- Redirect short URL to original URL.
- Scale: 100M URLs created/day.

### Design

```
[Client] → [API Server] → [DB: short_code → long_url]
                        ↑
                    [Cache: Redis]
```

**Encoding:** Generate random 6-character code from `[a-zA-Z0-9]` = 62^6 ≈ 56 billion combinations.

**Redirect:** HTTP 301 (permanent, cached by browser) or 302 (temporary, always hits server).

**Database:** Key-value store (Redis/DynamoDB) — simple lookup.

**Collision handling:** Check if code exists; regenerate if so.

---

## 2. Rate Limiter

### Requirements
- Limit each user to N requests per minute.
- Return 429 status when limit exceeded.

### Design

```
[Client] → [Rate Limiter Middleware] → [API Server]
                ↓
           [Redis: user_id → {count, window_start}]
```

**Algorithm:** Sliding window counter in Redis.

```
MULTI
INCR rate:user123
EXPIRE rate:user123 60
EXEC
```

---

## 3. Design Twitter Feed / News Feed

### Requirements
- Users post tweets.
- Users see timeline of people they follow.
- Scale: 300M DAU, 500M tweets/day.

### Design

```
Write Path:
[Post Tweet] → [Tweet Service] → [DB: tweets]
                               → [Fanout Service] → [Feed Cache: user_id → [tweet_ids]]

Read Path:
[Load Feed] → [Feed Service] → [Feed Cache] → merge, hydrate with tweet data → [Client]
```

**Fanout on write (push model):** On post, push tweet_id to all followers' feed caches. Fast reads, expensive writes for celebrities.

**Fanout on read (pull model):** On load, fetch tweets from all followees. Slow reads, cheap writes.

**Hybrid:** Push for users with few followers; pull for celebrities.

---

## 4. Design a Key-Value Store

### Requirements
- `put(key, value)`, `get(key)`, `delete(key)`.
- High availability and horizontal scalability.

### Design

**Consistent Hashing:** Distribute keys across nodes; minimize rehashing when nodes join/leave.

```
[Client] → [Coordinator Node] 
              → hash(key) → find responsible node(s) on ring
              → replicate to N nodes (e.g., N=3)
              → quorum (W=2 write quorum, R=2 read quorum)
```

**Conflict resolution:** Vector clocks or "last write wins" (timestamp).

**Gossip protocol:** Nodes share state information to detect failures.

---

## 5. Design a Chat System (e.g., WhatsApp)

### Requirements
- 1:1 and group messaging.
- Message delivery and read receipts.
- Online presence.

### Design

```
[User A] ──WebSocket──► [Chat Server] ──► [Message Queue (Kafka)]
                                                ↓
                                     [Message Delivery Service]
                                                ↓
                              ──WebSocket──► [User B]

Storage:
- [Messages DB: Cassandra] — wide-column, high write throughput
- [Media: S3 + CDN]
- [User/Group metadata: MySQL]
- [Presence: Redis]
```

**WebSocket:** Persistent connection for real-time bidirectional messaging.

**Message storage:** Cassandra with partitioning by `(user_id, month)` for efficient retrieval.

---

## 6. Design a Distributed Cache

### Requirements
- In-memory key-value store.
- High availability, low latency.
- Eviction when capacity reached.

### Design

```
[Client] → [Cache Cluster]
             [Node 1] [Node 2] [Node 3]
             
Consistent hashing → routes key to correct node
Replication factor = 2 → each key on 2 nodes for HA
```

**Eviction:** LRU using a doubly linked list + hash map.

**Persistence:** Optional append-only log (like Redis AOF).

---

## System Design Interview Checklist

- [ ] Clarify requirements (functional + non-functional)
- [ ] Estimate scale (users, reads/writes per second, storage)
- [ ] Define APIs
- [ ] High-level architecture diagram
- [ ] Data model / schema
- [ ] Deep dive on critical components
- [ ] Address bottlenecks (caching, sharding, replication)
- [ ] Discuss trade-offs
