## **Advanced observability and distributed tracing (Prometheus + Grafana + Jaeger)**

**Objective:** Extend your existing containerized web app to implement end-to-end observability with custom metrics, log correlation, and distributed tracing using OpenTelemetry, Prometheus, and Jaeger.

---

### **Instructions**

- Reuse the same app (Node.js/Express or Python/Flask) and ECS/EC2 runtime from previous projects

- Add OpenTelemetry instrumentation for HTTP server, HTTP client, and DB calls (if any), exporting traces to Jaeger

- Add RED metrics (Rate, Errors, Duration) and expose /metrics for Prometheus to scrape

- Include trace_id and span_id in structured JSON logs; ensure CloudWatch or Loki captures them

- Create a Grafana dashboard for latency, error rate, CPU/memory, and trace links

- Define alert rules: error rate above 5% or latency above 300ms for 10 minutes triggers an alert

- Validate by simulating load and errors, then confirm alert → trace → log correlation

- Clean up test routes but retain dashboards and alert configurations

---

### **Deliverables**

- App code with OpenTelemetry instrumentation

- Prometheus configuration file (prometheus.yml)

- Grafana dashboard JSON export

- Jaeger setup configuration

- Screenshots of Grafana dashboards, Jaeger traces, and correlated logs

- A 2-page report mapping symptom → trace → root cause

---

### **Submission requirements**

| **Item** | **Format** | **Contents** |
| --- | --- | --- |
| GitHub repository | All config files and evidence | App code, OTel config, prometheus.yml, Grafana JSON, Jaeger config, screenshots, 2-page report |
| Tools evidence | Version output or screenshot | OpenTelemetry SDK, Prometheus, Grafana, Jaeger, CloudWatch/Loki (optional) |
| Correlation evidence | Screenshots | Alert → trace → log correlation showing root cause identification |

---

### **Required file structure**

```
project/
├── app.py                    # or app.js — with OTel instrumentation
├── otel_config.py            # or otel_config.js
├── prometheus.yml
├── docker-compose.yml        # app, Prometheus, Grafana, Jaeger, Node Exporter
├── grafana/
│   └── dashboard.json
├── jaeger/
│   └── jaeger-config.yml
├── report.pdf                # 2-page observability report
└── screenshots/
    ├── grafana_dashboard.png
    ├── alert_triggered.png
    ├── jaeger_trace.png
    ├── log_correlation.png
    └── load_simulation.png
```

---

### **OpenTelemetry instrumentation example (Python/Flask)**

```python
# otel_config.py
from opentelemetry import trace
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
from opentelemetry.exporter.jaeger.thrift import JaegerExporter
from opentelemetry.instrumentation.flask import FlaskInstrumentor
from opentelemetry.instrumentation.requests import RequestsInstrumentor

def configure_tracing(app):
    jaeger_exporter = JaegerExporter(
        agent_host_name="jaeger",
        agent_port=6831,
    )

    provider = TracerProvider()
    provider.add_span_processor(BatchSpanProcessor(jaeger_exporter))
    trace.set_tracer_provider(provider)

    FlaskInstrumentor().instrument_app(app)
    RequestsInstrumentor().instrument()
```

---

### **RED metrics example (Python/Flask)**

```python
# app.py
from prometheus_client import Counter, Histogram, generate_latest
from flask import Flask, Response
import time

app = Flask(__name__)

REQUEST_COUNT = Counter(
    'http_requests_total',
    'Total HTTP requests',
    ['method', 'endpoint', 'status']
)

REQUEST_LATENCY = Histogram(
    'http_request_duration_seconds',
    'HTTP request latency',
    ['method', 'endpoint']
)

@app.before_request
def start_timer():
    from flask import g
    g.start_time = time.time()

@app.after_request
def record_metrics(response):
    from flask import g, request
    duration = time.time() - g.start_time
    REQUEST_COUNT.labels(
        method=request.method,
        endpoint=request.path,
        status=response.status_code
    ).inc()
    REQUEST_LATENCY.labels(
        method=request.method,
        endpoint=request.path
    ).observe(duration)
    return response

@app.route('/metrics')
def metrics():
    return Response(generate_latest(), mimetype='text/plain')
```

---

### **Structured JSON logging with trace correlation**

```python
import logging
import json
from opentelemetry import trace

class TraceContextFilter(logging.Filter):
    def filter(self, record):
        span = trace.get_current_span()
        ctx = span.get_span_context()
        record.trace_id = format(ctx.trace_id, '032x') if ctx.is_valid else ''
        record.span_id  = format(ctx.span_id, '016x') if ctx.is_valid else ''
        return True

def get_logger(name):
    logger = logging.getLogger(name)
    logger.addFilter(TraceContextFilter())
    handler = logging.StreamHandler()
    handler.setFormatter(logging.Formatter(
        '{"time": "%(asctime)s", "level": "%(levelname)s", '
        '"message": "%(message)s", '
        '"trace_id": "%(trace_id)s", "span_id": "%(span_id)s"}'
    ))
    logger.addHandler(handler)
    return logger
```

---

### **Prometheus configuration**

```yaml
# prometheus.yml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

rule_files:
  - alert_rules.yml

scrape_configs:
  - job_name: 'web-app'
    static_configs:
      - targets: ['<app-host>:5000']

  - job_name: 'node-exporter'
    static_configs:
      - targets: ['<ec2-host>:9100']
```

---

### **Alert rules**

```yaml
# alert_rules.yml
groups:
  - name: app_alerts
    rules:

      - alert: HighErrorRate
        expr: >
          rate(http_requests_total{status=~"5.."}[5m])
          / rate(http_requests_total[5m]) > 0.05
        for: 10m
        labels:
          severity: critical
        annotations:
          summary: "High error rate detected"
          description: "Error rate has exceeded 5% for more than 10 minutes."

      - alert: HighLatency
        expr: >
          histogram_quantile(0.95,
            rate(http_request_duration_seconds_bucket[5m])
          ) > 0.3
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "High latency detected"
          description: "P95 latency has exceeded 300ms for more than 10 minutes."
```

---

### **Docker Compose setup**

```yaml
# docker-compose.yml
services:

  app:
    build: .
    ports:
      - "5000:5000"
    environment:
      - OTEL_EXPORTER_JAEGER_AGENT_HOST=jaeger

  prometheus:
    image: prom/prometheus:latest
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - ./alert_rules.yml:/etc/prometheus/alert_rules.yml
    ports:
      - "9090:9090"

  grafana:
    image: grafana/grafana:latest
    ports:
      - "3000:3000"
    volumes:
      - ./grafana/dashboard.json:/etc/grafana/provisioning/dashboards/dashboard.json

  jaeger:
    image: jaegertracing/all-in-one:latest
    ports:
      - "6831:6831/udp"
      - "16686:16686"

  node-exporter:
    image: prom/node-exporter:latest
    ports:
      - "9100:9100"
```

---

### **Grafana dashboard — key panels**

| **Panel** | **Metric / Query** |
| --- | --- |
| Request rate (RPS) | rate(http_requests_total[5m]) |
| Error rate | rate(http_requests_total{status=~"5.."}[5m]) / rate(http_requests_total[5m]) |
| P95 latency | histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m])) |
| CPU usage | rate(process_cpu_seconds_total[5m]) |
| Memory usage | process_resident_memory_bytes |
| Trace links | Jaeger data source — link via trace_id from logs |

---

### **Validation steps**

- Simulate load using a tool like hey or locust:

```bash
hey -n 1000 -c 50 http://<EC2-IP>:5000/
```

- Simulate errors by hitting a route that returns 500 responses

- Confirm in Grafana that the error rate alert fires after 10 minutes above threshold

- Open Jaeger at http://<EC2-IP>:16686 and find the trace corresponding to the error

- Cross-reference the trace_id from the Jaeger trace with the structured JSON logs in CloudWatch or Loki

- Document the full chain: alert → trace → log → root cause

---

### **2-page report structure**

**Page 1 — Observability summary:**

- RED metrics collected and what they reveal about app behaviour

- Grafana dashboard panels and their purpose

- Alerts configured and conditions that triggered them

- Load simulation approach and results

**Page 2 — Distributed tracing and root cause analysis:**

- How OpenTelemetry was instrumented in the app

- Example of alert → trace → log correlation with screenshots

- Root cause identified from trace spans and correlated logs

- Recommendations for improving the observability setup

---

### **Tips**

- Install the OpenTelemetry auto-instrumentation package for your framework to minimize manual code changes (opentelemetry-instrumentation-flask or @opentelemetry/auto-instrumentations-node)

- Always propagate the traceparent header in outbound HTTP calls so traces span across services

- Use Jaeger's search feature to filter by error=true to quickly find failed traces during validation

- Correlate logs to traces by copying the trace_id from Jaeger and searching for it in CloudWatch Logs Insights

- Retain your dashboard.json export before cleanup so dashboards can be restored quickly in future projects
