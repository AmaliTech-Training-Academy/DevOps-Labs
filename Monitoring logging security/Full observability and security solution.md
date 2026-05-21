## **Full observability and security solution**

**Objective:** Extend the existing containerized web application to implement a complete observability and security stack using Prometheus, Grafana, and AWS services (CloudWatch, CloudTrail, and GuardDuty).

---

### **Instructions**

- Use the existing containerized app from the CI/CD project, ensuring it exposes metrics at /metrics

- Provision Prometheus on a new EC2 instance or container and configure it to scrape metrics from the app and Node Exporter

- Deploy Grafana connected to Prometheus and create dashboards for latency, requests per second, and error rates

- Configure alerts for high error rates (above 5%) using Grafana or Prometheus alerting rules

- Enable CloudWatch Logs and configure Docker to stream container logs to CloudWatch

- Enable AWS CloudTrail and GuardDuty for account activity monitoring and threat detection

- Store CloudTrail logs in an S3 bucket with encryption and lifecycle policies

- Verify that monitoring, alerts, and security detections are all working

- Clean up monitoring EC2 instances and AWS resources after verification

---

### **Deliverables**

- Prometheus configuration file (prometheus.yml)

- Grafana dashboard JSON export

- Screenshots of dashboards and triggered alerts

- CloudWatch and GuardDuty outputs

- A 2-page report summarizing insights

---

### **Submission requirements**

| **Item** | **Format** | **Contents** |
| --- | --- | --- |
| GitHub repository | All configuration files and evidence | prometheus.yml, Grafana dashboard JSON, screenshots, 2-page report |
| Tools evidence | Version output or screenshot | Prometheus, Grafana, AWS CloudWatch, CloudTrail, GuardDuty |
| Dashboard evidence | Screenshots | Functional dashboards showing latency, RPS, and error rates |
| Alert evidence | Screenshot | Alerts triggered for error rate above 5% |
| AWS evidence | Screenshots | CloudWatch logs, CloudTrail events, GuardDuty findings recorded |

---

### **Required file structure**

```
project/
├── prometheus.yml
├── grafana/
│   └── dashboard.json
├── docker-compose.yml        # includes app, Prometheus, Grafana, Node Exporter
├── cloudwatch/
│   └── awslogs.conf
├── report.pdf                # 2-page observability and security report
└── screenshots/
    ├── grafana_dashboard.png
    ├── alert_triggered.png
    ├── cloudwatch_logs.png
    ├── cloudtrail_events.png
    └── guardduty_findings.png
```

---

### **Prometheus configuration example**

```yaml
# prometheus.yml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'web-app'
    static_configs:
      - targets: ['<app-host>:5000']

  - job_name: 'node-exporter'
    static_configs:
      - targets: ['<ec2-host>:9100']
```

---

### **Grafana alert rule example**

```yaml
# High error rate alert (above 5%)
alert: HighErrorRate
expr: rate(http_requests_total{status=~"5.."}[5m])
      / rate(http_requests_total[5m]) > 0.05
for: 2m
labels:
  severity: critical
annotations:
  summary: "High error rate detected"
  description: "Error rate is above 5% for more than 2 minutes."
```

---

### **CloudWatch Docker logging configuration**

```ini
# /etc/awslogs/awslogs.conf
[general]
state_file = /var/lib/awslogs/agent-state

[/var/log/docker]
log_group_name  = /ec2/docker/containers
log_stream_name = {instance_id}
file            = /var/log/docker
```

---

### **AWS services checklist**

| **Service** | **Configuration required** |
| --- | --- |
| CloudWatch Logs | Docker log driver set to awslogs; log group created |
| CloudTrail | Trail enabled; logs stored in S3 bucket with encryption |
| GuardDuty | Enabled in AWS Console; findings reviewed |
| S3 (CloudTrail logs) | Encryption enabled; lifecycle policy to archive after 30 days |

---

### **2-page report structure**

**Page 1 — Observability summary:**

- Overview of metrics collected (latency, RPS, error rates)

- Description of dashboards created in Grafana

- Summary of alerts configured and any that were triggered

- Key insights from the monitoring data

**Page 2 — Security summary:**

- CloudTrail events recorded during the lab

- GuardDuty findings (if any) and their severity

- CloudWatch log highlights from container activity

- Recommendations for improving the security posture

---

### **Tips**

- Expose Prometheus metrics from your Flask or Node.js app using the official client library (prometheus_client for Python, prom-client for Node.js)

- Run Node Exporter as a container alongside your app in Docker Compose for easy setup

- Use Grafana's import feature to load pre-built dashboards from grafana.com as a starting point, then customize them

- Test your alert rules by deliberately introducing errors into your app to confirm the alert fires correctly

- Enable S3 bucket versioning alongside the lifecycle policy to protect CloudTrail logs from accidental deletion

- Never disable GuardDuty between verification and cleanup — findings may take a few minutes to appear
