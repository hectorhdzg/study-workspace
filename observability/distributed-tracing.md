# Distributed Tracing

## Why Distributed Tracing?

In a monolith, a stack trace shows you the whole picture. In microservices, a single user request may hit 10+ services. **Distributed tracing** connects the dots.

```
Without tracing:                     With tracing:
"Something is slow"                  "Payment API call takes 800ms 
 ¯\_(ツ)_/¯                           in Order Service span #4"
```

---

## Trace Anatomy

```
Trace ID: 4bf92f3577b34da6a3ce929d0e0e4736
│
├── Span: API Gateway (250ms)
│   span_id: 1, parent: none
│   attributes: http.method=POST, http.url=/api/orders, http.status_code=201
│
├── Span: Auth Service (20ms)
│   span_id: 2, parent: 1
│   attributes: auth.method=JWT, auth.result=success
│
├── Span: Order Service (200ms)
│   span_id: 3, parent: 1
│
│   ├── Span: PostgreSQL Query (15ms)
│   │   span_id: 4, parent: 3
│   │   attributes: db.system=postgresql, db.statement="INSERT INTO orders..."
│   │
│   ├── Span: Payment Service (150ms)  ← bottleneck
│   │   span_id: 5, parent: 3
│   │   attributes: http.method=POST, http.url=https://pay.stripe.com/v1/charges
│   │
│   └── Span: Kafka Produce (5ms)
│       span_id: 6, parent: 3
│       attributes: messaging.system=kafka, messaging.destination=order-events
```

---

## Context Propagation

For traces to work across services, the **trace context** must be passed along with each request. This is called **context propagation**.

### W3C Trace Context (Standard)

```
HTTP Header:
traceparent: 00-4bf92f3577b34da6a3ce929d0e0e4736-00f067aa0ba902b7-01
              │  │                                  │                  │
              │  trace-id (128-bit)                 span-id (64-bit)  flags (sampled)
              version
```

### How It Flows

```
[Service A]                    [Service B]                 [Service C]
 Create trace                   Extract header              Extract header
 Generate span 1                Create span 2               Create span 3
 Set traceparent header ──►     Set parent = span 1  ──►    Set parent = span 2
                                Set traceparent header       Report span 3
                                Report span 2
 Report span 1

All 3 spans share the same trace-id → reconstructed into a single trace
```

### Code Examples

```python
# Python (OpenTelemetry auto-instrumentation handles this automatically)
# Manual propagation if needed:
from opentelemetry import trace, context
from opentelemetry.propagate import inject, extract

# Inject trace context into outgoing request headers
headers = {}
inject(headers)  # adds traceparent header
requests.get("http://service-b/api", headers=headers)

# Extract trace context from incoming request
ctx = extract(request.headers)
with trace.get_tracer(__name__).start_as_current_span("my-operation", context=ctx):
    # This span is linked to the parent trace
    process_request()
```

```javascript
// Node.js (OpenTelemetry auto-instrumentation handles HTTP propagation)
const { trace, context, propagation } = require('@opentelemetry/api');

// Manual: inject into outgoing headers
const headers = {};
propagation.inject(context.active(), headers);
fetch('http://service-b/api', { headers });
```

```csharp
// .NET (OpenTelemetry auto-instruments HttpClient automatically)
// Activity is .NET's built-in span concept
using var activity = ActivitySource.StartActivity("ProcessOrder");
activity?.SetTag("order.id", orderId);

// HttpClient automatically propagates trace context
var response = await httpClient.GetAsync("http://service-b/api");
```

---

## Sampling Strategies

In high-throughput systems, tracing every request is expensive. **Sampling** reduces the volume while preserving visibility.

| Strategy | Description | Trade-off |
|----------|-------------|-----------|
| **Head-based** | Decide at the start of the trace (e.g., sample 10%) | Simple, but may miss interesting traces |
| **Tail-based** | Decide after the trace completes (keep errors, slow requests) | Better coverage, requires trace collector |
| **Rate-limiting** | Keep N traces per second | Predictable cost |
| **Priority** | Always sample errors, slow requests; downsample healthy ones | Best balance, more complex |

```python
# OpenTelemetry: sample 10% of traces
from opentelemetry.sdk.trace.sampling import TraceIdRatioBased
sampler = TraceIdRatioBased(0.1)  # 10% sampling rate
```

**Rule of thumb:** In production, 1-10% head-based sampling + 100% tail-based for errors/slow requests.

---

## Span Attributes (Semantic Conventions)

OpenTelemetry defines standard attribute names:

| Attribute | Example Value |
|-----------|--------------|
| `http.method` | `GET` |
| `http.url` | `https://api.example.com/users` |
| `http.status_code` | `200` |
| `db.system` | `postgresql` |
| `db.statement` | `SELECT * FROM users WHERE id = $1` |
| `messaging.system` | `kafka` |
| `messaging.destination` | `order-events` |
| `rpc.system` | `grpc` |
| `exception.message` | `NullPointerException` |

---

## Tracing Tools

| Tool | Type | Strengths |
|------|------|-----------|
| **Jaeger** | Open-source | CNCF project, good for Kubernetes |
| **Zipkin** | Open-source | Simple, lightweight |
| **Grafana Tempo** | Open-source | Cost-efficient, integrates with Grafana |
| **Datadog APM** | Commercial | Full-featured, great UI |
| **Azure Application Insights** | Commercial | Azure-native, auto-instrumentation |
| **AWS X-Ray** | Commercial | AWS-native |

---

## Common Interview Questions

- "How would you debug a slow API response in a microservices architecture?"
  → Distributed tracing: find the trace, identify the slowest span

- "How do you trace across async boundaries (message queues)?"
  → Inject trace context into message headers; extract on the consumer side

- "What's the trade-off with 100% sampling?"
  → High storage cost and network overhead; use tail-based sampling instead
