# Service Discovery & Coordination

In a distributed system with many services running across many nodes, services need to **find each other** (discovery) and **agree on shared state** (coordination). This page covers the tools and patterns for both.

---

## The Discovery Problem

In a dynamic environment (containers, auto-scaling, rolling deploys), services can't hardcode the addresses of their dependencies. IP addresses change as instances scale up/down, crash, or redeploy. Service discovery solves this by maintaining a live registry of available service instances.

---

## Service Discovery Patterns

### Client-Side Discovery

The client queries a service registry to get a list of available instances, then chooses one (using its own load-balancing logic).

```
Client → [Service Registry] → returns [Instance A:8080, Instance B:8081]
Client → [Instance A:8080]   (client picks one)
```

**Pros:** No extra network hop; client can implement smart load balancing.
**Cons:** Every client language/framework needs discovery logic.

**Used by:** Netflix Eureka + Ribbon, gRPC client-side balancing

### Server-Side Discovery

The client sends requests to a load balancer or router, which queries the registry and forwards the request.

```
Client → [Load Balancer / API Gateway] → [Service Registry]
                                        → [Instance B:8081]
```

**Pros:** Clients are simple (just call the load balancer); works with any language.
**Cons:** Extra network hop; the load balancer is a potential bottleneck.

**Used by:** AWS ALB + ECS, Kubernetes Services, Consul + Envoy

### DNS-Based Discovery

Services register with a DNS server that returns SRV or A records pointing to healthy instances. Clients resolve the DNS name to get instance addresses.

**Pros:** Universal — every language supports DNS.
**Cons:** DNS caching can serve stale records; TTLs must be short; limited health-check integration.

**Used by:** Consul DNS interface, AWS Route 53, Kubernetes CoreDNS

---

## Service Registry Tools

### ZooKeeper

A distributed coordination service originally built for Hadoop. Provides a hierarchical key-value store (like a file system) with strong consistency guarantees via Zab consensus.

**Use cases:** Leader election, distributed locking, configuration management, service discovery.

**Key concepts:**
- **Znodes** — Named data nodes arranged in a tree. Can be persistent or ephemeral.
- **Ephemeral nodes** — Automatically deleted when the session that created them expires. Perfect for service registration: if a service dies, its znode disappears.
- **Watches** — Clients register callbacks on znodes; they're notified on changes. Enables reactive service discovery.
- **Sequential nodes** — Auto-numbering for implementing distributed queues and locks.

**Trade-offs:** Strong consistency (linearizable reads/writes), but operationally complex. A ZooKeeper cluster (3-5 nodes) must be maintained separately from your application.

### etcd

A distributed key-value store that uses Raft for consensus. The backbone of Kubernetes — stores all cluster state.

**Key features:**
- **Strong consistency** — Linearizable reads and writes via Raft.
- **Watch API** — Stream changes to keys, enabling reactive discovery.
- **Leases** — Time-based keys that expire automatically (similar to ZooKeeper ephemeral nodes).
- **Transactions** — Atomic compare-and-swap operations for distributed locking.

**Compared to ZooKeeper:** Simpler API (flat key-value vs hierarchy), easier to operate, built-in gRPC interface. Preferred for new projects.

### Consul

HashiCorp's service mesh and discovery tool. Combines a service registry with health checking and a built-in DNS interface.

**Key features:**
- **Service registration** — Services register themselves or are registered by orchestrators.
- **Health checks** — HTTP, TCP, gRPC, and script-based checks. Unhealthy instances are automatically removed from queries.
- **Multi-datacenter** — Native support for service discovery across datacenters via WAN gossip.
- **Key-value store** — For configuration management and feature flags.
- **Connect (service mesh)** — Sidecar proxies for mTLS, traffic routing, and observability.

---

## Service Mesh

A service mesh adds a **dedicated infrastructure layer** for handling service-to-service communication. Instead of each service implementing retries, circuit breakers, mTLS, and observability, a sidecar proxy handles it transparently.

### Sidecar Pattern

Every service instance gets a proxy running alongside it (the "sidecar"). All inbound and outbound traffic flows through the sidecar. The application code doesn't know the proxy exists.

```
┌──────────────────────────┐     ┌──────────────────────────┐
│ Pod A                     │     │ Pod B                     │
│ ┌─────────┐ ┌───────────┐│     │┌───────────┐ ┌─────────┐ │
│ │ Service  │→│  Sidecar  ││────→││  Sidecar  │→│ Service │ │
│ │  App     │ │  Proxy    ││     ││  Proxy    │ │  App    │ │
│ └─────────┘ └───────────┘│     │└───────────┘ └─────────┘ │
└──────────────────────────┘     └──────────────────────────┘
```

### What the Mesh Provides

| Feature | Without Mesh | With Mesh |
|---------|-------------|-----------|
| mTLS encryption | Each service implements TLS | Automatic, zero-code |
| Retries & timeouts | Application-level libraries | Configured via mesh policy |
| Circuit breaking | Application-level (Hystrix, Polly) | Proxy-level, language-agnostic |
| Observability | Instrument each service | Automatic metrics, traces, logs |
| Traffic splitting | Custom routing logic | Declarative (e.g., 90/10 canary) |
| Access control | Application-level auth | Policy-based (service-to-service RBAC) |

### Popular Service Meshes

| Mesh | Proxy | Data Plane | Control Plane | Notes |
|------|-------|-----------|--------------|-------|
| Istio | Envoy | Sidecar | istiod | Most feature-rich; complex |
| Linkerd | linkerd2-proxy (Rust) | Sidecar | Control plane pods | Simpler, lighter than Istio |
| Consul Connect | Envoy (or built-in) | Sidecar | Consul servers | Integrates with HashiCorp ecosystem |
| AWS App Mesh | Envoy | Sidecar | AWS managed | Tight AWS integration |

---

## Distributed Locking

Sometimes multiple services need exclusive access to a shared resource. Distributed locks coordinate this.

### Redlock (Redis-based)

1. Try to acquire the lock on N (typically 5) independent Redis instances.
2. The lock is considered acquired if you succeed on a majority (N/2 + 1) within a timeout.
3. If acquired, the lock has a TTL — it auto-releases if the holder crashes.
4. To release, delete the lock key on all instances.

**Caveats:** Martin Kleppmann's critique points out that Redlock isn't safe under clock skew, GC pauses, or network delays. For correctness-critical locks, use fencing tokens with a consensus-based system (etcd, ZooKeeper).

### Consensus-Based Locks

etcd and ZooKeeper provide safe distributed locks:
- etcd: Use a lease + compare-and-swap transaction.
- ZooKeeper: Create a sequential ephemeral node; the lowest-numbered node holds the lock.

These are safe because the underlying consensus protocol ensures agreement even under failures.

---

## Interview Tips

- Know the difference between client-side and server-side discovery, and when to use each.
- Be able to explain the sidecar pattern and what problems a service mesh solves.
- ZooKeeper vs etcd: etcd is simpler and Kubernetes-native; ZooKeeper is battle-tested in Hadoop/Kafka ecosystems.
- Distributed locking is a common design question — know the Redlock algorithm and its limitations.

---

## Practice Questions

- Design a service discovery system for a microservice platform with 100+ services. How would you handle service health checking and stale registrations?
- Compare ZooKeeper, etcd, and Consul for a new Kubernetes-based platform.
- Why is distributed locking harder than it seems? What can go wrong with a simple Redis-based lock?
- Your company is adopting a service mesh. What benefits would you highlight, and what's the operational cost?
