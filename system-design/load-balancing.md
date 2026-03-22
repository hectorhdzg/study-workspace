# Load Balancing & Scalability

## Load Balancer

A **load balancer** distributes incoming traffic across multiple servers to:
- Prevent any single server from becoming a bottleneck.
- Provide high availability (if one server fails, traffic routes to others).
- Enable horizontal scaling.

---

## Load Balancing Algorithms

| Algorithm              | Description                                          | Best For                         |
|-----------------------|------------------------------------------------------|----------------------------------|
| **Round Robin**        | Requests distributed evenly in order                 | Homogeneous servers              |
| **Weighted Round Robin**| Servers with higher capacity get more requests      | Heterogeneous servers            |
| **Least Connections**  | Route to server with fewest active connections       | Long-lived connections           |
| **IP Hash**            | Same client always routes to same server             | Session persistence (sticky)     |
| **Random**             | Random server selection                              | Simple, roughly even load        |

---

## L4 vs L7 Load Balancing

| Layer | Level     | What it sees    | Examples                      |
|-------|-----------|-----------------|-------------------------------|
| L4    | Transport | IP + TCP/UDP    | AWS NLB, HAProxy (TCP mode)   |
| L7    | Application| HTTP/HTTPS headers, cookies, URL | AWS ALB, Nginx, Envoy |

**L7 enables:** URL-based routing, SSL termination, content-based routing, A/B testing.

---

## High Availability Patterns

### Active-Active

All instances are live and serve traffic simultaneously.

```
[Client] → [Load Balancer] → [Server A] (active)
                            → [Server B] (active)
```

### Active-Passive

One instance is live; standby takes over on failure.

```
[Client] → [Load Balancer] → [Server A] (active)
                            → [Server B] (standby - heartbeat check)
```

---

## Scaling Strategies

### Horizontal Scaling (Scale Out)
Add more servers behind a load balancer.

```
v1: [LB] → [Server 1]
v2: [LB] → [Server 1] [Server 2] [Server 3]
```

### Vertical Scaling (Scale Up)
Upgrade a single server to larger instance (more CPU, RAM).

### Auto-Scaling
Automatically add/remove servers based on metrics (CPU, request rate).
- **AWS:** EC2 Auto Scaling Groups, ECS service auto-scaling.
- **GCP:** Managed Instance Groups.

---

## Reverse Proxy

A **reverse proxy** sits in front of servers and handles:
- SSL/TLS termination.
- Compression.
- Caching.
- Rate limiting.
- Load balancing.

**Common tools:** Nginx, HAProxy, Envoy, Traefik.

---

## Rate Limiting

Protects your system from abuse and ensures fair usage.

### Algorithms

| Algorithm          | Description                                           |
|-------------------|-------------------------------------------------------|
| **Token Bucket**   | Tokens added at fixed rate; each request costs 1 token|
| **Leaky Bucket**   | Requests queued and processed at fixed rate           |
| **Fixed Window**   | Count requests per time window (e.g., 100/minute)     |
| **Sliding Window** | Rolling window to prevent boundary bursts             |

### Token Bucket (Simple JS)

```javascript
class TokenBucket {
  constructor(capacity, refillRate) {
    this.capacity = capacity;
    this.tokens = capacity;
    this.refillRate = refillRate; // tokens per second
    this.lastRefill = Date.now();
  }

  consume(tokens = 1) {
    this._refill();
    if (this.tokens >= tokens) {
      this.tokens -= tokens;
      return true; // allowed
    }
    return false; // rate limited
  }

  _refill() {
    const now = Date.now();
    const elapsed = (now - this.lastRefill) / 1000;
    this.tokens = Math.min(this.capacity, this.tokens + elapsed * this.refillRate);
    this.lastRefill = now;
  }
}
```

```python
import time

class TokenBucket:
    def __init__(self, capacity, refill_rate):
        self.capacity = capacity
        self.tokens = capacity
        self.refill_rate = refill_rate  # tokens per second
        self.last_refill = time.time()

    def consume(self, tokens=1):
        self._refill()
        if self.tokens >= tokens:
            self.tokens -= tokens
            return True   # allowed
        return False      # rate limited

    def _refill(self):
        now = time.time()
        elapsed = now - self.last_refill
        self.tokens = min(self.capacity, self.tokens + elapsed * self.refill_rate)
        self.last_refill = now
```

```csharp
public class TokenBucket
{
    private readonly int _capacity;
    private readonly double _refillRate;
    private double _tokens;
    private DateTime _lastRefill;

    public TokenBucket(int capacity, double refillRate)
    {
        _capacity = capacity;
        _refillRate = refillRate;
        _tokens = capacity;
        _lastRefill = DateTime.UtcNow;
    }

    public bool Consume(int tokens = 1)
    {
        Refill();
        if (_tokens >= tokens) { _tokens -= tokens; return true; }
        return false;
    }

    private void Refill()
    {
        var now = DateTime.UtcNow;
        var elapsed = (now - _lastRefill).TotalSeconds;
        _tokens = Math.Min(_capacity, _tokens + elapsed * _refillRate);
        _lastRefill = now;
    }
}
```

---

## DNS & Global Load Balancing

- **DNS round robin** — Multiple A records for the same domain.
- **GeoDNS** — Routes users to nearest data center based on IP.
- **Anycast** — Same IP announced from multiple locations; BGP routes to nearest.
