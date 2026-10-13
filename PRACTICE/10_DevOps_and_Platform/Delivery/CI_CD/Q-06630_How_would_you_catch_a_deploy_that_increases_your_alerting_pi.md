---
type: "practice"
difficulty: "Deep"
---

# How would you catch a deploy that increases your alerting pipeline's end-to-end detection latency (ingest to page) beyond your SLO, before it hits on-call in prod?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Infrastructure Drift Detection](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Infrastructure_Drift_Detection.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this challenge, we need to proactively monitor and catch any deployments that increase our alerting pipeline's end-to-end detection latency (from ingestion to a page being sent) beyond acceptable Service Level Objectives (SLOs). This involves integrating robust monitoring throughout the CI/CD process. We'll walk through setting up real-time anomaly detection, establishing baselines, and configuring automated alerts before production deployments.

First, let’s triage the question:

What is the Question Probing?: The interviewer wants to see how you can integrate comprehensive monitoring and alerting mechanisms into your CI/CD pipeline.
Key Assumptions: You have a functioning CI/CD pipeline with access to detailed metrics at various stages (e.g., ingestion, processing, storage, notifications).
Goals: Identify any deployment that might increase latency beyond predefined thresholds.

Now let's delve into the solution:

Step 1: Establish Real-Time Anomaly Detection

We will leverage real-time anomaly detection tools like Prometheus' alertmanager and Grafana’s alerting capabilities to monitor our SLOs in near real-time. For example, we can set up a dashboard that visualizes latency metrics from ingestion to page.

MARKDOWN
| Metric | Explanation |
|--------|-------------|
| Ingestion Time | Time taken for data to be received by the system |
| Processing Time | Time spent processing ingested data |
| Storage Time | Time data spends in storage before being available for analysis |
| Notification Time | Time from when a page is generated until it reaches on-call personnel |
Step 2: Define Baselines and Thresholds

Using historical data, we can establish baselines for our SLOs. For instance, if the average end-to-end detection latency should not exceed 5 minutes, this becomes our threshold.

Ingestion
Processing
Storage
Notification
End-to-End Latency
Step 3: Implement Automated Alerts

We will configure automated alerts to trigger when the end-to-end detection latency exceeds our predefined thresholds. This involves setting up alert rules in Prometheus:

YAML
groups:
  - name: ci-cd-latency
    rules:
      - alert: EndToEndLatencySLOExceeded
        expr: histogram_quantile(0.95, increase(end_to_end_latency[1m])) > 300
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "End-to-end detection latency exceeded SLO."
Step 4: Integrate with CI/CD Pipeline

We will integrate these alerting mechanisms into our CI/CD pipeline. This means that any deployment candidate must pass these checks before proceeding to production.

PYTHON
def check_slo_before_deployment(deployment):
    # Fetch current latency metrics
    current_latency = fetch_latency_metrics()

    # Check against SLOs
    if not all(current_latency <= thresholds for thresholds in SLO_THRESHOLD_MAP.values()):
        raise Exception("Latency exceeded SLO, deployment blocked.")

    return "Deployment allowed"
Step 5: Post-Deployment Monitoring and Feedback Loops

Even after a successful deployment, we will continuously monitor the latency metrics to ensure they remain within acceptable limits. This helps in identifying any degradation early.

#### Closing Summary

By implementing real-time anomaly detection, defining clear SLO baselines, configuring automated alerts, integrating these checks into our CI/CD pipeline, and maintaining post-deployment monitoring, we can catch deployments that increase end-to-end detection latency before they impact on-call personnel.

#### Common Mistakes/Follow-Ups
Ignoring Baseline Establishment: Failing to establish accurate historical baselines can lead to false positives or missed alerts.
Over-reliance on Alerts: Relying too heavily on alerting mechanisms without understanding the root causes of latency issues.
Lack of Continuous Monitoring Post-Deployment: Without ongoing monitoring, potential issues might go unnoticed until they become critical.

**Comparison:**

```
2023-12-03
2023-12-10
2023-12-17
2023-12-24
2023-12-31
2024-01-07
2024-01-14
2024-01-21
2024-01-28
2024-02-04
2024-02-11
2024-02-18
2024-02-25
setup
baseline_estimation
integration
pre_deployment_monitoring
post_deployment_monitoring
feedback_loops
Pre-deployment Checks
Deployment Monitoring
Post-deployment Feedback
CI/CD Pipeline with Latency Monitoring
```

**Code:**

```yaml
# Prometheus Alert Rule Example
groups:
  - name: ci-cd-latency
    rules:
      - alert: EndToEndLatencySLOExceeded
        expr: histogram_quantile(0.95, increase(end_to_end_latency[1m])) > 300
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "End-to-end detection latency exceeded SLO."
```

**Example input / output:**

```text
# Example input/output for check_slo_before_deployment function
def fetch_latency_metrics():
    return {"ingestion_time": 10, "processing_time": 20, "storage_time": 30, "notification_time": 40}

SLO_THRESHOLD_MAP = {
    "ingestion_time": 5,
    "processing_time": 15,
    "storage_time": 25,
    "end_to_end_latency": 60
}

deployment = check_slo_before_deployment(fetch_latency_metrics())
print(deployment) # Should return "Deployment allowed" if all metrics are within SLO.
```

**Data:**

```
MARKDOWN
Copy
| Metric | Description |
|--------|-------------|
| Ingestion Time | Data reception time |
| Processing Time | Time spent processing data |
| Storage Time | Time data spends in storage before analysis |
| Notification Time | Time from page generation to on-call notification |
```

**Flow:**

1. Ingestion
2. Processing
3. Storage
4. Notification
5. End-to-End Latency
6. Check SLO
7. Allow Deployment
8. Audit and Feedback

