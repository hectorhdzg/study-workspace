# Message Queues

## Why Message Queues?

Message queues **decouple** producers and consumers, enabling:
- **Asynchronous processing** — Producers don't wait for consumers.
- **Load leveling** — Absorb traffic spikes; process at a steady rate.
- **Reliability** — Messages persist until acknowledged.
- **Fan-out** — One message delivered to multiple consumers.

---

## Core Concepts

| Term        | Description                                               |
|------------|-----------------------------------------------------------|
| Producer   | Publishes messages to a queue or topic                    |
| Consumer   | Reads and processes messages                              |
| Queue      | Point-to-point: one consumer per message                  |
| Topic      | Pub/Sub: message broadcast to all subscribers             |
| Ack        | Consumer acknowledges successful processing               |
| Dead Letter Queue (DLQ) | Messages that failed processing repeatedly   |

---

## Point-to-Point vs Pub/Sub

```
Point-to-Point:
  [Producer] → [Queue] → [Consumer A]  (only one consumer gets each message)

Pub/Sub:
  [Producer] → [Topic] → [Consumer A]  (all subscribers get each message)
                       → [Consumer B]
                       → [Consumer C]
```

---

## Popular Message Queue Technologies

| System       | Model       | Strengths                                     |
|-------------|-------------|-----------------------------------------------|
| RabbitMQ    | Queue/Topic | Rich routing, low latency                      |
| Apache Kafka| Log/Topic   | High throughput, replay, durable, ordered      |
| AWS SQS     | Queue       | Managed, scalable, simple                      |
| AWS SNS     | Pub/Sub     | Fan-out to SQS, Lambda, email, SMS             |
| Redis Pub/Sub| Pub/Sub    | Fast, in-memory, no persistence                |
| Google Pub/Sub| Topic     | Managed, global, at-least-once delivery        |

---

## Kafka Deep Dive

Kafka stores messages as an **immutable log** in ordered topics partitioned across brokers.

```
Topic: "orders"
  Partition 0: [msg0, msg1, msg2, ...]
  Partition 1: [msg0, msg1, msg2, ...]

Consumer Group A reads from partition 0
Consumer Group B reads from partition 1 (parallel processing)
Consumer Group C reads ALL partitions (different group = independent offset)
```

**Key features:**
- **Retention:** Messages kept for a configurable period (not deleted on consume).
- **Replay:** Consumers can re-read messages by resetting offset.
- **Ordering:** Guaranteed within a partition; use partition key for related messages.
- **Throughput:** Millions of messages/second.

---

## Patterns

### Event-Driven Architecture

```
[Order Service] --OrderPlaced--> [Kafka Topic: orders]
                                    → [Payment Service]
                                    → [Inventory Service]
                                    → [Notification Service]
```

### Saga Pattern (Distributed Transactions)

Break a distributed transaction into a sequence of local transactions, each publishing an event to trigger the next step. On failure, compensating transactions undo previous steps.

---

## When to Use a Message Queue

- Long-running tasks (email sending, PDF generation, video encoding).
- Decoupling microservices.
- Handling traffic spikes (queue absorbs burst).
- Event sourcing and audit logs.
- Fan-out notifications.
