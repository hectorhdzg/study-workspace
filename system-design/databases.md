# Databases

## Relational vs NoSQL

| Feature           | Relational (SQL)            | NoSQL                          |
|------------------|-----------------------------|---------------------------------|
| Schema            | Fixed, predefined           | Flexible / schema-less          |
| Query language    | SQL (structured)            | Varies (document, key-value, etc.)|
| ACID              | Yes                         | Often eventual consistency      |
| Scaling           | Vertical (+ read replicas)  | Horizontal                      |
| Best for          | Structured data, complex queries | High scale, flexible data  |
| Examples          | PostgreSQL, MySQL, SQLite   | MongoDB, DynamoDB, Cassandra    |

---

## NoSQL Types

| Type         | Description                              | Examples               |
|-------------|------------------------------------------|------------------------|
| Key-Value   | Simple map of key → value                | Redis, DynamoDB        |
| Document    | JSON-like documents, nested structures   | MongoDB, Firestore     |
| Column-family| Rows with dynamic columns, wide tables  | Cassandra, HBase       |
| Graph       | Nodes and edges for relationships        | Neo4j, Amazon Neptune  |

---

## Indexing

An **index** is a data structure (usually a B-Tree or Hash) that speeds up lookups at the cost of extra storage and slower writes.

```sql
-- Create an index on email for fast lookups
CREATE INDEX idx_users_email ON users(email);

-- Composite index (covers queries on both columns)
CREATE INDEX idx_orders ON orders(user_id, created_at);
```

**When to index:**
- Columns frequently used in `WHERE`, `JOIN`, `ORDER BY`, `GROUP BY`.
- Foreign key columns.
- High cardinality columns (many unique values).

**When NOT to index:**
- Small tables.
- Columns rarely used in queries.
- Tables with high write frequency (indexes slow down writes).

---

## Database Replication

**Primary-Replica (Master-Slave):**
- Writes go to primary.
- Reads distributed across replicas.
- Provides read scalability and redundancy.

**Multi-Primary (Multi-Master):**
- All nodes accept writes.
- Requires conflict resolution.
- Higher write throughput, more complexity.

---

## Database Sharding

**Sharding** partitions data across multiple databases to scale writes.

| Strategy          | Description                              | Pros / Cons                         |
|------------------|------------------------------------------|--------------------------------------|
| Range-based       | Shard by value range (e.g., user ID 1-1M on shard 1)| Simple, hot spots possible |
| Hash-based        | `shard = hash(key) % num_shards`         | Even distribution, hard to rebalance|
| Directory-based   | Lookup service maps key → shard          | Flexible, single point of failure   |
| Geographic        | Shard by region                          | Low latency, data locality          |

**Challenges:** Cross-shard joins, global transactions, rebalancing.

---

## SQL vs NoSQL Decision Guide

**Use SQL when:**
- Data is structured and relational.
- You need ACID transactions (e.g., financial systems).
- Complex queries with joins and aggregations.

**Use NoSQL when:**
- Massive scale (100M+ users).
- Unstructured or variable data.
- Need flexible schema evolution.
- Write-heavy workloads at scale.
- Simple access patterns (no complex joins).

---

## Common Database Patterns

### Connection Pooling
Reuse database connections instead of creating a new one per request.

### Query Optimization
- Use `EXPLAIN` to analyze query execution plan.
- Avoid `SELECT *` — fetch only needed columns.
- Use pagination (`LIMIT`/`OFFSET` or cursor-based).

### Caching Query Results
Cache frequently read data in Redis/Memcached. Invalidate on write (cache-aside or write-through).
