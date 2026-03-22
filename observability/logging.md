# Logging

## Structured vs Unstructured Logging

```
❌ Unstructured:
2026-03-21 14:32:01 ERROR Payment failed for user 456, amount $99.99, card declined

✅ Structured (JSON):
{
  "timestamp": "2026-03-21T14:32:01.123Z",
  "level": "ERROR",
  "service": "payment-service",
  "trace_id": "abc123",
  "span_id": "def456",
  "message": "Payment failed",
  "user_id": "u_456",
  "amount": 99.99,
  "error_code": "card_declined"
}
```

**Why structured?** You can filter, aggregate, and search by any field. "Show me all ERROR logs from payment-service where amount > 50 in the last hour."

---

## Log Levels

| Level | When to Use | Example |
|-------|-------------|---------|
| **DEBUG** | Detailed diagnostic info (disabled in production) | Variable values, loop iterations |
| **INFO** | Normal operations worth recording | Request received, job started, deployment complete |
| **WARN** | Something unexpected but handled | Retry succeeded, cache miss, deprecated API used |
| **ERROR** | Operation failed, needs attention | Payment declined, DB connection lost, unhandled exception |
| **FATAL** | System is going down | Out of memory, critical config missing |

**Production defaults:** INFO and above. Set DEBUG per-service temporarily when investigating issues.

---

## Logging in Code

### Python

```python
import structlog

log = structlog.get_logger()

# Automatically includes timestamp, level, and any bound context
log.info("order_created", order_id="ORD-123", amount=99.99, user_id="u_456")
# {"event": "order_created", "order_id": "ORD-123", "amount": 99.99, ...}

log.error("payment_failed", order_id="ORD-123", error="card_declined")
```

### JavaScript

```javascript
const pino = require('pino');
const log = pino({ level: 'info' });

log.info({ orderId: 'ORD-123', amount: 99.99, userId: 'u_456' }, 'order_created');
// {"level":30,"time":1711024321123,"orderId":"ORD-123","msg":"order_created"}

log.error({ orderId: 'ORD-123', err: error }, 'payment_failed');
```

### C# (.NET)

```csharp
using Microsoft.Extensions.Logging;

// Structured logging with message templates
logger.LogInformation("Order created: {OrderId}, Amount: {Amount}", orderId, amount);
// Renders as: "Order created: ORD-123, Amount: 99.99"
// But stores OrderId and Amount as structured fields for querying

logger.LogError(exception, "Payment failed for {OrderId}", orderId);

// Using Serilog (popular structured logging library)
Log.Information("Order {OrderId} created for {Amount:C}", orderId, amount);
```

---

## Correlation: Connecting Logs to Traces

Include `trace_id` in every log entry so you can jump from a log line to its full trace.

```python
import structlog
from opentelemetry import trace

def add_trace_context(logger, method_name, event_dict):
    span = trace.get_current_span()
    ctx = span.get_span_context()
    if ctx.is_valid:
        event_dict["trace_id"] = format(ctx.trace_id, '032x')
        event_dict["span_id"] = format(ctx.span_id, '016x')
    return event_dict

structlog.configure(processors=[add_trace_context, ...])
```

```csharp
// .NET + Serilog: automatically enriches logs with trace context
// when using OpenTelemetry
Log.Logger = new LoggerConfiguration()
    .Enrich.WithProperty("ServiceName", "order-service")
    .Enrich.FromLogContext()  // includes Activity (trace) context
    .WriteTo.Console(new JsonFormatter())
    .CreateLogger();
```

---

## Log Aggregation Architecture

```
[Service A] ─┐
[Service B] ─┤──► [Log Shipper]  ──►  [Log Store]  ──►  [Query UI]
[Service C] ─┘    (Fluentd/        (Elasticsearch/    (Kibana/
                   Filebeat/         Loki/Splunk)       Grafana)
                   Vector)
```

| Stack | Components | Best For |
|-------|-----------|----------|
| **ELK** | Elasticsearch + Logstash + Kibana | Full-text search, rich analysis |
| **Grafana + Loki** | Loki (log store) + Grafana | Cost-efficient, label-based, Grafana ecosystem |
| **Splunk** | All-in-one | Enterprise, compliance |
| **CloudWatch Logs** | AWS-native | AWS workloads |
| **Azure Monitor Logs** | Log Analytics workspace + KQL | Azure workloads |

---

## Logging Best Practices

| Do | Don't |
|----|-------|
| Use structured logging (JSON) | Log plain text strings |
| Include correlation IDs (trace_id) | Log without context |
| Log at the right level | Log everything at INFO |
| Redact PII (emails, SSNs, tokens) | Log passwords or secrets |
| Set log retention policies | Keep logs forever (cost!) |
| Sample verbose logs in production | Log every DB query at INFO |
| Include actionable context | Log "error occurred" with no details |

---

## Log Retention & Cost

| Environment | Retention | Reasoning |
|-------------|-----------|-----------|
| Development | 3-7 days | Short-lived, high volume |
| Staging | 14-30 days | Debugging pre-prod issues |
| Production (DEBUG) | Don't enable | Cost explosion |
| Production (INFO+) | 30-90 days | Operational needs |
| Audit / Compliance | 1-7 years | Regulatory requirements |

**Cost tip:** Logs are often the most expensive part of observability. Use sampling for high-volume endpoints, and drop DEBUG logs in production.
