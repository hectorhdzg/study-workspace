# Caching

## Why Cache?

Caching stores frequently accessed data in fast storage (memory) to:
- Reduce database load.
- Lower response latency (memory ~1000x faster than disk).
- Increase throughput.

---

## Cache Strategies

### Cache-Aside (Lazy Loading)

Application manages the cache explicitly.

```
Read:  Check cache → cache miss? → read DB → populate cache → return
Write: Update DB → invalidate cache entry
```

**Pros:** Only cache what's actually needed. Cache failure doesn't break the system.
**Cons:** First request is slow (cache miss). Data can be stale until TTL expires.

### Write-Through

Write to cache and DB together on every write.

```
Write: Update cache → update DB → return
Read:  Check cache → return (always warm)
```

**Pros:** Cache always up to date.
**Cons:** Write latency (two operations per write). May cache data never read.

### Write-Behind (Write-Back)

Write to cache immediately; asynchronously write to DB.

```
Write: Update cache → return immediately → async write to DB
```

**Pros:** Very low write latency.
**Cons:** Risk of data loss if cache crashes before DB write.

### Read-Through

Cache sits in front of DB and loads data on miss automatically.

**Similar to cache-aside** but cache is responsible for loading from DB.

---

## Cache Eviction Policies

| Policy | Description                               |
|--------|-------------------------------------------|
| **LRU** | Evict Least Recently Used              |
| **LFU** | Evict Least Frequently Used             |
| **FIFO**| Evict oldest inserted item              |
| **TTL** | Expire after a set time-to-live         |

---

## Cache Levels

| Level          | Technology               | Latency  |
|---------------|--------------------------|----------|
| CPU cache      | L1/L2/L3                | ns       |
| In-process     | Local memory (e.g., Map) | ~100 ns  |
| Distributed    | Redis, Memcached          | ~1 ms    |
| CDN            | Cloudflare, Akamai       | ~10 ms   |

---

## Redis Overview

Redis is an in-memory data store supporting rich data structures.

```
# Key-value
SET user:1 "Alice"
GET user:1

# Hash
HSET user:1 name "Alice" email "alice@example.com"
HGET user:1 name

# List
LPUSH queue task1 task2
RPOP queue

# Set
SADD active_users user:1 user:2
SMEMBERS active_users

# Sorted Set (score-based)
ZADD leaderboard 100 user:1 200 user:2
ZRANGE leaderboard 0 -1 WITHSCORES REV
```

---

## CDN (Content Delivery Network)

A CDN caches static assets (images, CSS, JS, videos) at edge nodes close to users.

**Flow:**
1. User requests `cdn.example.com/image.jpg`.
2. CDN edge node checks local cache.
3. **Cache hit:** Serve directly.
4. **Cache miss:** Fetch from origin server, cache, serve.

**Use CDN for:** Static assets, large media files, public API responses.

---

## Cache Invalidation Strategies

- **TTL (Time-to-Live):** Expire after N seconds. Simple but may serve stale data.
- **Event-driven invalidation:** Invalidate on write events (Pub/Sub, webhooks).
- **Versioned keys:** `user:1:v3` — new version on update; old versions naturally expire.

---

## Common Caching Problems

| Problem          | Description                                                       | Solution                       |
|-----------------|-------------------------------------------------------------------|--------------------------------|
| Cache Stampede  | Many requests hit DB simultaneously on cache expiry               | Mutex lock, probabilistic refresh|
| Cache Avalanche | Many caches expire at the same time                              | Random TTL jitter              |
| Cache Penetration| Query for non-existent data bypasses cache each time             | Cache null values, Bloom filter|
