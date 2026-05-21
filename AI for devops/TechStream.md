## **The "Self-Healing" system — TechStream**

**Scenario:** TechStream wants to reduce their Mean Time To Resolution (MTTR). They want a system that not only monitors their application but automatically detects anomalies and attempts to fix them before waking up an engineer.

---

### **Objectives**

- Implement "Golden Signal" monitoring

- Simulate an incident and trigger an automated remediation (self-healing)

- Use AI/ML for root cause analysis (simulated)

---

### **Instructions**

#### **1. Monitoring stack**

Deploy an application (e.g., a buggy web server) and set up monitoring (CloudWatch or Prometheus/Grafana). Dashboard the Golden Signals:

- Latency

- Traffic

- Errors

- Saturation

#### **2. Anomaly injection**

Create a "Chaos Script" that artificially spikes CPU or generates HTTP 500 errors on the web server.

#### **3. Alerting and remediation**

- Configure a CloudWatch Alarm (or Prometheus Alert) that triggers when Error Rate > 5%

- Connect this alarm to an EventBridge rule

- Target a Lambda Function or SSM Run Command that automatically restarts the web server service or scales the ASG out

#### **4. AI analysis**

Enable Amazon DevOps Guru (or similar trial) on the stack. Trigger the chaos script again and export the DevOps Guru insight showing the anomaly correlation.
