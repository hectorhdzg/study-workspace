# Core Concepts

## What is Observability?

**Observability** is the ability to understand the internal state of a system by examining its external outputs. It answers: _"Why is the system behaving this way?"_

**Monitoring** tells you _something is wrong_. **Observability** helps you figure out _why_.

---

## The Three Pillars

```
                    Observability
                    /     |     \
               Logs    Metrics   Traces
              (what)   (how much) (where)
```

| Pillar | What It Captures | Best For |
|--------|-----------------|----------|
| **Logs** | Discrete events with context | Debugging specific errors, audit trails |
| **Metrics** | Numeric measurements over time | Dashboards, alerting, trends |
| **Traces** | Request flow across services | Latency analysis, dependency mapping |

---

## Logs

Timestamped records of discrete events.

```json
{
  "timestamp": "2026-03-21T14:32:01.123Z",
  "level": "ERROR",
  "service": "payment-service",
  "trace_id": "abc123",
  "message": "Payment failed",
  "user_id": "u_456",
  "error": "Card declined",
  "amount": 99.99,
  "currency": "USD"
}
```

**Key principles:**
- Use **structured logging** (JSON) — not plain text
- Include **correlation IDs** (trace_id, request_id) for cross-service debugging
- Log at appropriate levels: DEBUG, INFO, WARN, ERROR
- Don't log sensitive data (PII, passwords, tokens)

---

## Metrics

Numeric values measured over time, aggregated into time series.

**Types:**
| Type | Description | Example |
|------|-------------|---------|
| **Counter** | Only goes up (monotonically increasing) | Total HTTP requests, errors |
| **Gauge** | Goes up and down | CPU usage, queue depth, active connections |
| **Histogram** | Distribution of values in buckets | Request duration (P50, P95, P99) |
| **Summary** | Similar to histogram, calculates quantiles client-side | Response size percentiles |

**Example (Prometheus format):**
```
# Counter
http_requests_total{method="GET", status="200"} 15234

# Gauge
node_memory_usage_bytes 1073741824

# Histogram
http_request_duration_seconds_bucket{le="0.1"} 2400
http_request_duration_seconds_bucket{le="0.5"} 4200
http_request_duration_seconds_bucket{le="1.0"} 4800
http_request_duration_seconds_count 5000
http_request_duration_seconds_sum 892.4
```

---

## Traces

A **trace** represents the full journey of a single request across services.

```
Trace: abc123
├── [API Gateway]        0ms ─────── 250ms
│   ├── [Auth Service]   10ms ── 30ms
│   └── [Order Service]  35ms ────── 240ms
│       ├── [DB Query]   40ms ── 60ms
│       ├── [Payment API] 65ms ──── 200ms  ← bottleneck!
│       └── [Cache Write] 205ms ─ 210ms
```

Each box is a **span** — a named, timed operation. Spans have:
- **Trace ID** — Groups all spans from the same request
- **Span ID** — Unique ID for this span
- **Parent Span ID** — Creates the tree structure
- **Duration** — How long this operation took
- **Attributes** — Key-value metadata (`http.method`, `db.statement`, etc.)

---

## The Golden Signals (Google SRE)

The four signals you should always monitor:

| Signal | Question | Metric Example |
|--------|----------|---------------|
| **Latency** | How long do requests take? | P50, P95, P99 response time |
| **Traffic** | How much demand is on the system? | Requests per second |
| **Errors** | What's the failure rate? | 5xx rate, error percentage |
| **Saturation** | How full is the system? | CPU %, memory %, queue depth |

---

## RED Method (for request-driven services)

| Metric | Description |
|--------|-------------|
| **R**ate | Requests per second |
| **E**rrors | Failed requests per second |
| **D**uration | Distribution of request durations |

## USE Method (for resources — CPU, memory, disk, network)

| Metric | Description |
|--------|-------------|
| **U**tilization | % of resource being used |
| **S**aturation | Work queued / waiting |
| **E**rrors | Error events on the resource |

---

## Observability Stack (Common Tools)

| Layer | Tools |
|-------|-------|
| **Metrics** | Prometheus, Grafana, Datadog, CloudWatch, Azure Monitor |
| **Logs** | ELK (Elasticsearch + Logstash + Kibana), Loki, Splunk, CloudWatch Logs |
| **Traces** | Jaeger, Zipkin, Tempo, Datadog APM, Azure Application Insights |
| **All-in-one** | Datadog, New Relic, Dynatrace, Grafana Cloud, Azure Monitor |
| **Instrumentation** | OpenTelemetry (vendor-neutral standard) |
