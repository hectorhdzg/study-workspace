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

---

## 7. Design an AI-Powered Customer Support Chatbot (RAG)

### Requirements
- Answer customer questions using company knowledge base (docs, FAQs, policies).
- Fallback to human agent when the bot can't answer.
- Support 10K concurrent users, < 3s response time.
- Cite sources in responses. Don't hallucinate.

### High-Level Architecture

```
[User] → [Web App / Chat Widget]
              ↓
         [API Gateway + Auth]
              ↓
         [Chat Service]
              ↓
    ┌─────────┴──────────┐
    │   RAG Pipeline      │
    │                     │
    │ 1. Embed query      │ → [Embedding Model API]
    │ 2. Search           │ → [Vector DB (Qdrant/Pinecone)]
    │ 3. Re-rank          │ → [Cross-Encoder / Cohere Rerank]
    │ 4. Generate answer  │ → [LLM API (GPT-4o / Claude)]
    └─────────┬──────────┘
              ↓
         [Response + Citations]
              ↓
    ┌─────────┴──────────┐
    │  Guardrails         │
    │  - Content filter   │
    │  - PII redaction    │
    │  - Confidence check │
    └─────────┬──────────┘
              ↓
         [User sees answer]
              │
         (low confidence?) → [Route to Human Agent via Queue]
```

### Data Ingestion Pipeline (Offline)

```
[Knowledge Sources]                  [Vector DB]
  ├─ Help articles (CMS)                 ↑
  ├─ FAQ pages                    [Embed chunks]
  ├─ Policy PDFs                         ↑
  └─ Product docs              [Chunk documents]
        ↓                                ↑
  [Document Loader]  →  [Chunker]  →  [Embedder]  →  [Index]

Triggered by: webhook on CMS update, nightly cron, or manual
```

### Key Components

**Chat Service**
- Stateless API that orchestrates the RAG pipeline.
- Stores conversation history in Redis (TTL 24h) for multi-turn context.
- Passes last N messages as context to LLM.

**Vector Database**
- Stores document chunks with embeddings.
- Metadata per chunk: `source_url`, `title`, `last_updated`, `category`.
- Use metadata filters: e.g., only search "billing" category for billing questions.

**LLM Integration**
- System prompt instructs: "Answer ONLY from the provided context. If unsure, say you'll connect to a human agent."
- Temperature = 0 for factual consistency.
- Streaming response for real-time UX.

**Guardrails**
- Content Safety API to block harmful responses.
- PII detection to redact before logging.
- Confidence scoring: if the model says "I'm not sure" or no relevant chunks found → route to human.

### APIs

```
POST /api/chat
Body: { "session_id": "abc", "message": "How do I return an item?" }
Response (streamed): {
  "reply": "You can return items within 30 days of purchase...",
  "sources": [
    {"title": "Return Policy", "url": "https://help.example.com/returns"}
  ],
  "confidence": 0.92
}

POST /api/feedback
Body: { "message_id": "xyz", "rating": "positive" }
```

### Database Schema

```sql
-- Conversations
CREATE TABLE conversations (
    id UUID PRIMARY KEY,
    user_id UUID,
    created_at TIMESTAMP,
    status VARCHAR(20)  -- 'active', 'resolved', 'escalated'
);

-- Messages
CREATE TABLE messages (
    id UUID PRIMARY KEY,
    conversation_id UUID REFERENCES conversations(id),
    role VARCHAR(10),       -- 'user', 'assistant', 'agent'
    content TEXT,
    sources JSONB,          -- [{"title": "...", "url": "..."}]
    confidence FLOAT,
    created_at TIMESTAMP
);
```

### Scale & Performance

| Concern | Solution |
|---------|----------|
| LLM latency (1-5s) | Stream tokens to client, cache frequent queries |
| Vector DB throughput | HNSW index, read replicas, pre-filter by metadata |
| Embedding cost | Batch embed during ingestion, cache query embeddings |
| Conversation state | Redis with TTL, not in the LLM context after 24h |
| Concurrent users | Stateless chat service, horizontal scaling behind LB |
| Knowledge freshness | Webhook-triggered re-indexing on content updates |

### Trade-offs Discussion

| Decision | Option A | Option B |
|----------|----------|----------|
| **LLM choice** | GPT-4o (smartest, $$$) | GPT-4o-mini (cheaper, faster, good enough for FAQ) |
| **Self-host vs API** | Self-host open LLM (control, privacy) | API (easy, no GPU infra) |
| **Chunking** | Small (200 tokens, precise) | Large (1000 tokens, more context) |
| **When to escalate** | Low confidence → human | Always show bot answer, offer escalation |
| **Hybrid search** | Vector only (simpler) | Vector + BM25 keyword (better recall) |
