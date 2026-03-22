# Metrics & Monitoring

## Metric Types Deep Dive

### Counter

Only goes up. Resets to zero on process restart.

```python
# Prometheus client (Python)
from prometheus_client import Counter

request_count = Counter(
    'http_requests_total',
    'Total HTTP requests',
    ['method', 'endpoint', 'status']
)

# Increment on each request
request_count.labels(method='GET', endpoint='/api/users', status='200').inc()
```

```csharp
// .NET (System.Diagnostics.Metrics)
var meter = new Meter("MyApp");
var requestCounter = meter.CreateCounter<long>("http.requests.total",
    description: "Total HTTP requests");

requestCounter.Add(1, new KeyValuePair<string, object?>("method", "GET"),
                      new KeyValuePair<string, object?>("status", "200"));
```

**Use for:** Request count, error count, bytes transferred, messages processed.

**Tip:** To get a _rate_ from a counter, use `rate()` in PromQL: `rate(http_requests_total[5m])` = requests/second over last 5 minutes.

### Gauge

Goes up and down. Represents a current value.

```python
from prometheus_client import Gauge

queue_depth = Gauge('queue_depth', 'Current queue depth', ['queue_name'])
queue_depth.labels(queue_name='orders').set(42)
queue_depth.labels(queue_name='orders').inc()  # 43
queue_depth.labels(queue_name='orders').dec()  # 42
```

```javascript
// OpenTelemetry (Node.js)
const meter = opentelemetry.metrics.getMeter('my-app');
const activeConnections = meter.createUpDownCounter('active_connections');
activeConnections.add(1);   // connection opened
activeConnections.add(-1);  // connection closed
```

**Use for:** CPU usage, memory usage, active connections, queue depth, temperature.

### Histogram

Measures the distribution of values. Crucial for latency monitoring.

```python
from prometheus_client import Histogram

request_duration = Histogram(
    'http_request_duration_seconds',
    'Request duration in seconds',
    ['endpoint'],
    buckets=[0.01, 0.05, 0.1, 0.25, 0.5, 1.0, 2.5, 5.0, 10.0]
)

# Record an observation
with request_duration.labels(endpoint='/api/users').time():
    handle_request()
```

```csharp
var requestDuration = meter.CreateHistogram<double>("http.request.duration",
    unit: "s", description: "Request duration");

var stopwatch = Stopwatch.StartNew();
HandleRequest();
requestDuration.Record(stopwatch.Elapsed.TotalSeconds,
    new KeyValuePair<string, object?>("endpoint", "/api/users"));
```

**Why histograms over averages?** Averages hide outliers. P99 = 5s means 1% of users wait 5+ seconds — that's thousands of users at scale.

---

## Percentiles (Why They Matter)

```
Requests:  [10ms, 12ms, 11ms, 15ms, 13ms, 14ms, 12ms, 11ms, 500ms, 10ms]

Average:   60.8ms  ← looks fine!
P50:       12ms    ← half of requests under 12ms
P95:       500ms   ← 5% of users see 500ms+
P99:       500ms   ← the tail matters at scale
```

| Percentile | Meaning | When to Use |
|-----------|---------|-------------|
| **P50** | Median — the "typical" experience | General performance |
| **P95** | 1 in 20 requests is slower | SLO target (commonly) |
| **P99** | 1 in 100 requests is slower | Tail latency, high-value users |
| **P99.9** | 1 in 1000 requests | Critical paths (payments) |

---

## Prometheus + Grafana Stack

The de facto open-source monitoring stack:

```
[Your App] ──metrics──► [Prometheus] ──query──► [Grafana]
  (exposes /metrics)     (scrapes + stores)     (dashboards + alerts)
```

### PromQL (Prometheus Query Language)

```promql
# Request rate (requests/second over 5 minutes)
rate(http_requests_total[5m])

# Error rate percentage
rate(http_requests_total{status=~"5.."}[5m])
/ rate(http_requests_total[5m]) * 100

# P95 latency
histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))

# CPU usage percentage
100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

# Top 5 endpoints by request rate
topk(5, sum by (endpoint) (rate(http_requests_total[5m])))
```

---

## Dashboard Best Practices

### The Four Golden Signals Dashboard

```
┌──────────────────────┬──────────────────────┐
│  Request Rate        │  Error Rate          │
│  ████████████░░░░    │  ░░░░░░░░░░░████     │
│  1,234 req/s         │  0.3% (SLO: <1%)     │
├──────────────────────┼──────────────────────┤
│  Latency (P50/P95)   │  Saturation           │
│  P50: 45ms           │  CPU: 62%            │
│  P95: 180ms          │  Memory: 78%         │
│  P99: 450ms          │  Disk I/O: 34%       │
└──────────────────────┴──────────────────────┘
```

**Tips:**
- Put the most important signals at the top
- Show SLO targets as horizontal threshold lines
- Use consistent time ranges across panels
- Include links to runbooks for common issues

---

## Cardinality

**Cardinality** = number of unique time series. High cardinality = expensive.

```
❌ Bad: http_requests_total{user_id="..."}     → millions of series
✅ Good: http_requests_total{endpoint="/api/users", method="GET", status="200"}
                                                → hundreds of series
```

| Label | Cardinality | Safe? |
|-------|------------|-------|
| `method` (GET, POST, ...) | ~10 | Yes |
| `status` (200, 404, 500) | ~10 | Yes |
| `endpoint` (/api/users) | ~100 | Yes |
| `user_id` | Millions | **No** — use logs instead |
| `request_id` | Unbounded | **No** — use traces instead |

**Rule of thumb:** Keep total series under 100K per service. Use logs/traces for high-cardinality debugging.

---

## Cloud-Native Monitoring

| Cloud | Metrics Service | Pre-built Dashboards |
|-------|----------------|---------------------|
| **AWS** | CloudWatch Metrics | EC2, RDS, Lambda dashboards |
| **Azure** | Azure Monitor Metrics | App Service, AKS, SQL dashboards |
| **GCP** | Cloud Monitoring | GKE, Cloud Run, BigQuery dashboards |
