---
type: "practice"
difficulty: "Deep"
---

# How would you catch a regression where a deploy quietly increased trace sampling drop rate and hid errors?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Anomaly and Forecast Monitors](<../../../../TOOLS/Datadog/Datadog_Platform/Anomaly_and_Forecast_Monitors.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of catching regressions in a deployed system that increases trace sampling drop rates, we need to ensure comprehensive monitoring and alerting mechanisms are in place. Datadog is an excellent tool for this purpose due to its powerful observability features and ability to detect anomalies. The goal here is to identify a situation where the trace sampling rate decreases, leading to fewer traces being collected and potential errors going unnoticed.

In essence, we need to monitor the trace sampling rate metrics in real-time, set up alerting rules, and continuously validate that error rates remain stable post-deployment. Let's break this down step by step from first principles.

Understanding the Problem: We are concerned about a scenario where a deploy increases the trace sampling drop rate, effectively reducing the number of traces being collected. This can lead to undetected errors and thus regressions in our application performance or behavior.

Metrics Collection and Monitoring:

Trace Sampling Rate Metrics: Collect metrics on the current and historical trace sampling rates using Datadog.
Error Rate Metrics: Monitor error rates, which will help us detect if there are more unreported errors due to reduced trace coverage.

Setting Up Alerting Rules:

Thresholds for Trace Sampling Rate: Define thresholds that trigger alerts when the trace sampling rate drops below a certain percentage. For example, an alert might be triggered if it falls below 80% of its previous value or historical average.
Error Rate Thresholds: Set up alerting on error rates to detect increases in errors that may have gone undetected due to reduced trace coverage.

Continuous Validation:

Post-Deployment Monitoring: After a deployment, closely monitor the metrics for any anomalies in both sampling rate and error rate.
Baseline Establishment: Establish baselines by collecting data before deploying changes so that you can compare post-deployment values against these baselines.

Detection of Regressions:

Use Datadog’s anomaly detection capabilities to automatically identify unusual patterns in the metrics. This will help catch issues early and provide actionable insights.

Actionable Insights:

When an alert is triggered, correlate this with logs and traces from Datadog to understand the root cause of any detected anomalies.
Implement rollback procedures if necessary based on these insights.

**Comparison:**

| Timestamp       | Sampling Rate (last hour) | Error Rate (last hour) |
|-----------------|--------------------------|-----------------------|
| 2023-10-01      | 0.7                      | 0.4                   |
| 2023-10-02      | 0.85                     | 0.6                   |

**Code:**

```python
# Example Python code snippet for setting up basic Datadog alerts (simplified)
import requests

def set_datadog_alert(sampling_rate_threshold, error_rate_threshold):
    # Define alert conditions
    sampling_rate_alert = {
        "name": "Low Trace Sampling Rate",
        "condition": {
            "query": f"avg(last_1h):trace.sample.rate{*sampling_rate_threshold}",
            "thresholds": [0.8, 0.95],
            "type": "sampled"
        },
        "alert_type": "warning",
        "notifications": ["email_alert@example.com"]
    }

    error_rate_alert = {
        "name": "Increased Error Rate",
        "condition": {
            "query": f"avg(last_1h):error.rate{*error_rate_threshold}",
            "thresholds": [0.5, 1],
            "type": "sampled"
        },
        "alert_type": "warning",
        "notifications": ["email_alert@example.com"]
    }

    # Send request to Datadog API to create alerts
    response = requests.post("https://api.datadoghq.com/api/v1/monitor", headers={
        "Content-Type": "application/json",
        "DD-API-KEY": "your_api_key"
    }, json=[sampling_rate_alert, error_rate_alert])

    return response.status_code

set_datadog_alert(0.8, 0.5)
```

**Example input / output:**

```text
Input:
sampling_rate_data = [
    {"timestamp": "2023-10-01T00:00:00Z", "rate": 0.7},
    {"timestamp": "2023-10-02T00:00:00Z", "rate": 0.85}
]
error_rate_data = [
    {"timestamp": "2023-10-01T00:00:00Z", "rate": 0.4},
    {"timestamp": "2023-10-02T00:00:00Z", "rate": 0.6}
]

Output:
{
  "sampling_rate_alerts": [
    {"timestamp": "2023-10-02T00:00:00Z", "status": "ALERT"}
  ],
  "error_rate_alerts": [
    {"timestamp": "2023-10-02T00:00:00Z", "status": "OK"}
  ]
}
```

**Data:**

```json
{
  "sampling_rate_data": [
    {"timestamp": "2023-10-01T00:00:00Z", "rate": 0.7},
    {"timestamp": "2023-10-02T00:00:00Z", "rate": 0.85}
  ],
  "error_rate_data": [
    {"timestamp": "2023-10-01T00:00:00Z", "rate": 0.4},
    {"timestamp": "2023-10-02T00:00:00Z", "rate": 0.6}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Initial Sampling Rate Baseline
6. Is Sampling Rate ThresholdExceeded?
7. Generate Alert for ReducedTrace Sampling
8. Check Error Rate Metrics
9. Is Error Rate Increased?
10. Notify Team of Potential Issues
11. Continue Monitoring

