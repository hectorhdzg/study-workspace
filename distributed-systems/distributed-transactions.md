# Distributed Transactions

A distributed transaction spans multiple nodes or services that must all succeed or all fail together. This is much harder than local transactions because any node can crash, messages can be lost, and there's no shared clock.

---

## The Problem

In a monolithic database, transactions are straightforward — the database engine handles ACID guarantees internally. In a distributed system:
- Data lives on different nodes (partitioned database) or different services (microservices).
- A failure after some nodes commit but before others do leaves the system in an inconsistent state.
- You can't just "rollback" a network message that was already sent.

---

## Two-Phase Commit (2PC)

The classic protocol for atomic commits across multiple nodes.

### How It Works

**Phase 1 — Prepare (Voting):**
1. The **coordinator** sends a `PREPARE` message to all participants.
2. Each participant executes the transaction locally (but doesn't commit) and replies `YES` (ready to commit) or `NO` (abort).

**Phase 2 — Commit/Abort (Decision):**
3. If **all** participants voted YES → coordinator sends `COMMIT`.
4. If **any** participant voted NO → coordinator sends `ABORT`.
5. Participants execute the decision and acknowledge.

```
Coordinator          Participant A       Participant B
    |--- PREPARE ------->|                    |
    |--- PREPARE -------------------------------->|
    |<-- YES ------------|                    |
    |<-- YES ----------------------------------|
    |--- COMMIT -------->|                    |
    |--- COMMIT --------------------------------->|
    |<-- ACK ------------|                    |
    |<-- ACK ----------------------------------|
```

### The Blocking Problem

2PC has a critical flaw: if the coordinator crashes **after sending PREPARE but before sending the decision**, participants are stuck. They've voted YES (locked their resources) but don't know whether to commit or abort. They must **block** until the coordinator recovers.

This is why 2PC is called a **blocking protocol** — a single coordinator failure can halt progress.

### When to Use 2PC

- Within a single organization's database cluster (e.g., distributed database like Spanner).
- When strong consistency is required and the coordinator is highly available.
- **Avoid** across microservices in different teams — too fragile and tightly coupled.

---

## Three-Phase Commit (3PC)

3PC adds an extra phase to reduce the blocking window. After PREPARE, the coordinator sends a `PRE-COMMIT` message before the final `COMMIT`. This lets participants know the decision before resources are locked.

In practice, 3PC is rarely used because it assumes bounded network delays (which real networks don't guarantee) and the extra round-trip adds latency. Most systems use 2PC with a highly available coordinator instead.

---

## Saga Pattern

Sagas are the practical alternative to distributed transactions in microservice architectures. Instead of locking resources across services, a saga breaks the transaction into a sequence of **local transactions**, each with a **compensating action** that undoes its effect if a later step fails.

### Choreography vs Orchestration

| Approach | How It Works | Pros | Cons |
|----------|-------------|------|------|
| Choreography | Each service listens for events and triggers the next step | Loose coupling, no single point of failure | Hard to track overall progress; complex failure chains |
| Orchestration | A central orchestrator tells each service what to do | Clear flow, easier monitoring | Orchestrator is a single point of failure; tighter coupling |

### Example: Order Processing Saga

```
Step 1: Order Service    → Create order          (compensate: cancel order)
Step 2: Payment Service  → Charge payment        (compensate: refund payment)
Step 3: Inventory Service → Reserve stock         (compensate: release stock)
Step 4: Shipping Service → Schedule delivery      (compensate: cancel shipment)
```

If Step 3 fails (out of stock):
```
→ Compensate Step 2: refund payment
→ Compensate Step 1: cancel order
```

### Saga Guarantees

- Sagas provide **eventual consistency**, not ACID isolation.
- Intermediate states are visible to other transactions (no isolation).
- Compensating actions must be **idempotent** — they may be retried on failure.
- Some actions can't be compensated (e.g., sending an email) — these are called **pivot transactions** and should be placed last.

---

## Exactly-Once Semantics

In a distributed system, messages can be lost or duplicated. The three delivery guarantees:

| Guarantee | Meaning | How |
|-----------|---------|-----|
| At-most-once | Message delivered 0 or 1 times | Fire and forget; no retries |
| At-least-once | Message delivered 1 or more times | Retry until acknowledged; may duplicate |
| Exactly-once | Message processed exactly 1 time | At-least-once delivery + idempotent processing |

True exactly-once delivery is impossible in theory (Two Generals Problem), but **exactly-once processing** is achievable by making operations idempotent:

**Idempotency keys** — Assign each request a unique ID. The receiver tracks processed IDs and skips duplicates.

```python
# Pseudocode: idempotent payment processing
def process_payment(request_id, amount):
    if already_processed(request_id):
        return get_cached_result(request_id)
    
    result = charge_card(amount)
    mark_processed(request_id, result)
    return result
```

**Transactional outbox** — Write the message and the business data in the same local transaction. A separate process reads the outbox table and publishes messages. This avoids the dual-write problem (business DB + message queue).

```
┌─────────────────────────┐
│  Local Transaction       │
│  1. Update order status  │
│  2. Insert into outbox   │
└─────────────────────────┘
         ↓
[Outbox Relay] → reads outbox → publishes to message queue
```

---

## Comparison

| Approach | Consistency | Availability | Complexity | Use Case |
|----------|------------|-------------|-----------|----------|
| 2PC | Strong (atomic) | Low (blocking) | Medium | Single-org distributed DB |
| 3PC | Strong (non-blocking in theory) | Medium | High | Rarely used in practice |
| Saga | Eventual | High | Medium-High | Microservices |
| Outbox + Idempotency | Eventual | High | Medium | Event-driven systems |

---

## Interview Tips

- 2PC comes up often — know the two phases, the blocking problem, and why it's unsuitable for microservices.
- Sagas are the standard answer for "how do you handle transactions across microservices?"
- Be ready to design a saga with compensating actions for a specific scenario.
- The outbox pattern is a strong answer for "how do you reliably publish events after a database write?"

---

## Practice Questions

- Design a saga for a travel booking system (flight + hotel + car rental).
- What happens in 2PC if the coordinator crashes after sending PREPARE but only one participant received COMMIT?
- How would you implement exactly-once payment processing in a system that uses Kafka?
- Compare the trade-offs of saga choreography vs orchestration for an e-commerce checkout flow.
