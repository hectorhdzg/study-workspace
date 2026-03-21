# System Design

## Topics

- [Core Concepts](./core-concepts.md)
- [Databases](./databases.md)
- [Caching](./caching.md)
- [Load Balancing & Scalability](./load-balancing.md)
- [Message Queues](./message-queues.md)
- [Common System Designs](./common-designs.md)

## Interview Approach

1. **Clarify requirements** — Ask about scale, features, constraints, and non-functional requirements (latency, availability, consistency).
2. **Estimate scale** — Calculate storage, bandwidth, and throughput needs.
3. **High-level design** — Draw a block diagram with major components.
4. **Deep dive** — Drill into specific components as directed.
5. **Identify bottlenecks** — Consider single points of failure, hot spots, and scaling limits.
6. **Trade-offs** — Discuss consistency vs. availability, cost vs. performance, etc.

## Scalability Checklist

- [ ] Stateless services (easy horizontal scaling)
- [ ] Database sharding or replication strategy
- [ ] Caching layer (CDN, in-memory, query cache)
- [ ] Asynchronous processing via message queues
- [ ] Load balancer in front of application servers
- [ ] Rate limiting and circuit breakers
- [ ] Monitoring, alerting, and logging
