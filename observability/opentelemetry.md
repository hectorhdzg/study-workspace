# OpenTelemetry

## What is OpenTelemetry?

**OpenTelemetry** (OTel) is the vendor-neutral, open-source standard for collecting telemetry data (traces, metrics, logs). It's a CNCF project and the second most active project after Kubernetes.

**Before OTel:** Each vendor (Datadog, New Relic, Jaeger, etc.) had its own SDK. Switching vendors meant changing all your instrumentation code.

**With OTel:** Instrument once with OTel, send data to any backend.

```
[Your Application]
       │
  [OTel SDK]  ← instrument once
       │
  [OTel Collector]  ← routes, processes, exports
       │
  ┌────┴─────────────┐
  │  Any backend:     │
  │  Jaeger           │
  │  Prometheus       │
  │  Datadog          │
  │  Azure Monitor    │
  │  Grafana Cloud    │
  └───────────────────┘
```

---

## Architecture

```
[Auto-Instrumentation]     [Manual Instrumentation]
  (HTTP, DB, gRPC              (custom spans,
   libraries detected           business metrics)
   automatically)                    │
         │                           │
         └──────────┬────────────────┘
                    │
              [OTel SDK]
              (Tracer + Meter + Logger Providers)
                    │
              [Exporters]
              (OTLP, Prometheus, Jaeger, Zipkin, Console)
                    │
         [OTel Collector] (optional but recommended)
              │         │
         [Backend A] [Backend B]
```

---

## Quick Start Examples

### Python

```python
# Install: pip install opentelemetry-distro opentelemetry-exporter-otlp
# Auto-instrument: opentelemetry-bootstrap -a install

from opentelemetry import trace, metrics
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
from opentelemetry.exporter.otlp.proto.grpc.trace_exporter import OTLPSpanExporter
from opentelemetry.sdk.metrics import MeterProvider
from opentelemetry.sdk.metrics.export import PeriodicExportingMetricReader
from opentelemetry.exporter.otlp.proto.grpc.metric_exporter import OTLPMetricExporter

# Setup tracing
trace.set_tracer_provider(TracerProvider())
trace.get_tracer_provider().add_span_processor(
    BatchSpanProcessor(OTLPSpanExporter(endpoint="http://localhost:4317"))
)

# Setup metrics
reader = PeriodicExportingMetricReader(OTLPMetricExporter(endpoint="http://localhost:4317"))
metrics.set_meter_provider(MeterProvider(metric_readers=[reader]))

# Use in your code
tracer = trace.get_tracer("my-service")
meter = metrics.get_meter("my-service")

request_counter = meter.create_counter("http.requests", description="Total requests")

@app.route("/api/orders")
def create_order():
    with tracer.start_as_current_span("create-order") as span:
        span.set_attribute("order.type", "standard")
        request_counter.add(1, {"method": "POST", "endpoint": "/api/orders"})
        # ... your logic
```

**Or just use auto-instrumentation (zero code changes):**

```bash
pip install opentelemetry-distro opentelemetry-exporter-otlp
opentelemetry-bootstrap -a install

# Run your app with automatic instrumentation
opentelemetry-instrument \
  --service_name my-service \
  --exporter_otlp_endpoint http://localhost:4317 \
  python app.py
```

### Node.js

```javascript
// tracing.js - load before your app
const { NodeSDK } = require('@opentelemetry/sdk-node');
const { OTLPTraceExporter } = require('@opentelemetry/exporter-trace-otlp-grpc');
const { OTLPMetricExporter } = require('@opentelemetry/exporter-metrics-otlp-grpc');
const { getNodeAutoInstrumentations } = require('@opentelemetry/auto-instrumentations-node');
const { PeriodicExportingMetricReader } = require('@opentelemetry/sdk-metrics');

const sdk = new NodeSDK({
  serviceName: 'my-service',
  traceExporter: new OTLPTraceExporter({ url: 'http://localhost:4317' }),
  metricReader: new PeriodicExportingMetricReader({
    exporter: new OTLPMetricExporter({ url: 'http://localhost:4317' }),
  }),
  instrumentations: [getNodeAutoInstrumentations()],
});

sdk.start();
```

```bash
# Run with auto-instrumentation
node --require ./tracing.js app.js
```

### .NET (C#)

```csharp
// Program.cs
using OpenTelemetry.Resources;
using OpenTelemetry.Trace;
using OpenTelemetry.Metrics;

var builder = WebApplication.CreateBuilder(args);

// Add OpenTelemetry
builder.Services.AddOpenTelemetry()
    .ConfigureResource(r => r.AddService("my-service"))
    .WithTracing(tracing => tracing
        .AddAspNetCoreInstrumentation()    // auto-instrument HTTP
        .AddHttpClientInstrumentation()     // auto-instrument outgoing HTTP
        .AddSqlClientInstrumentation()      // auto-instrument SQL
        .AddOtlpExporter(o => o.Endpoint = new Uri("http://localhost:4317")))
    .WithMetrics(metrics => metrics
        .AddAspNetCoreInstrumentation()
        .AddHttpClientInstrumentation()
        .AddOtlpExporter(o => o.Endpoint = new Uri("http://localhost:4317")));

var app = builder.Build();

// Custom spans
var activitySource = new ActivitySource("my-service");

app.MapPost("/api/orders", async (OrderRequest req) =>
{
    using var activity = activitySource.StartActivity("CreateOrder");
    activity?.SetTag("order.type", req.Type);
    // ... your logic
});
```

---

## OTel Collector

The **Collector** is a standalone process that receives, processes, and exports telemetry data. Recommended for production.

```yaml
# otel-collector-config.yaml
receivers:
  otlp:
    protocols:
      grpc:
        endpoint: 0.0.0.0:4317
      http:
        endpoint: 0.0.0.0:4318

processors:
  batch:
    timeout: 5s
    send_batch_size: 1024
  memory_limiter:
    limit_mib: 512

exporters:
  otlp/jaeger:
    endpoint: jaeger:4317
    tls:
      insecure: true
  prometheus:
    endpoint: 0.0.0.0:8889

service:
  pipelines:
    traces:
      receivers: [otlp]
      processors: [memory_limiter, batch]
      exporters: [otlp/jaeger]
    metrics:
      receivers: [otlp]
      processors: [memory_limiter, batch]
      exporters: [prometheus]
```

**Why use a Collector?**
- Decouples apps from backends — switch backends without code changes
- Batching, retry, and memory management
- Sampling and filtering before export
- Single point to configure auth/TLS to backends

---

## What Gets Auto-Instrumented?

| Language | HTTP | Database | gRPC | Messaging | Framework |
|----------|------|----------|------|-----------|-----------|
| **Python** | requests, urllib, aiohttp | psycopg2, SQLAlchemy, pymongo | grpcio | kafka, celery | Flask, Django, FastAPI |
| **Node.js** | http, fetch, axios | pg, mysql, mongoose, redis | @grpc/grpc-js | kafkajs, amqplib | Express, Fastify, Nest |
| **.NET** | HttpClient | SqlClient, EF Core, StackExchange.Redis | Grpc.Net.Client | MassTransit | ASP.NET Core |
| **Java** | HttpURLConnection, OkHttp | JDBC, Hibernate | gRPC | Kafka, RabbitMQ | Spring Boot, JAX-RS |

---

## OTLP (OpenTelemetry Protocol)

The native wire protocol for OTel. Supports gRPC (port 4317) and HTTP/protobuf (port 4318).

All major backends now accept OTLP directly:
- Grafana Cloud, Datadog, New Relic, Honeycomb, Dynatrace, Azure Monitor, AWS CloudWatch
