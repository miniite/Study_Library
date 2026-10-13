---
type: "practice"
difficulty: "Deep"
---

# Watchdog flagged an anomaly that on-call dismissed as a false positive three times this week; how do you decide whether to tune it or turn it off?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Anomaly and Forecast Monitors](<../../../../TOOLS/Datadog/Datadog_Platform/Anomaly_and_Forecast_Monitors.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with recurring false positives from Datadog's Watchdog, deciding whether to tune or turn off an alert requires a careful analysis of both the alert's behavior and the impact on your operations. A deep understanding of the principles behind anomaly detection is crucial here. In this scenario, we will evaluate the nature of the alerts, their frequency, and the potential consequences of false positives versus missed signals.

Firstly, it’s important to understand that anomaly detection in monitoring systems like Datadog works by setting thresholds for normal behavior based on historical data. These thresholds are designed to flag significant deviations from expected patterns, which could indicate issues that need attention. However, these systems aren't always perfect and can generate false positives due to various factors such as noise in the data or unexpected changes.

#### Triage
What the Question is Really Asking

The question asks how we should handle a situation where an alert has been dismissed as a false positive multiple times but continues to trigger regularly. This involves assessing whether the alert is genuinely noisy, or if it’s signaling something important that needs tuning for better accuracy.

#### Assumptions
and Key Considerations
Frequency of Alerts: The fact that this anomaly has triggered three times in one week suggests there might be a pattern worth investigating.
Impact on Operations: Dismissing an alert as a false positive could lead to missing actual issues if the underlying condition is significant.
Cost vs Benefit: Tuning or turning off alerts impacts resource allocation; accurate alerts are crucial for timely resolution.

#### First-Principles Solution

Review Recent Logs and Metrics:

Step 1: Collect detailed logs from the time periods when the anomaly triggered to identify any unusual events that might correlate with the alert.
Step 2: Check if there were any recent changes in infrastructure, software deployments, or configuration updates that could have caused the false positives.

Analyze Historical Data:

Step 3: Compare the current thresholds with historical data to understand how they are performing over time.
Step 4: Use Datadog’s built-in analytics and visualization tools to create time-series graphs of relevant metrics around the alert times.

Tune or Refine the Alert:

Step 5: If there is a clear pattern, consider adjusting the threshold or adding more context-based filters.
Step 6: For example, if the anomaly only occurs during specific hours when other processes are running, add a time-of-day filter to reduce false positives.

Evaluate Impact of False Positives:

Step 7: Assess whether missing these alerts could have significant operational impacts (e.g., service degradation, security breaches).
Step 8: Consider implementing stricter alerting policies or adding more complex anomaly detection algorithms that can better handle noisy data.

#### Cons
ider Alternative Actions:

Step 9: If tuning the alert is not feasible, consider setting up a more robust on-call rotation to ensure that alerts are promptly reviewed.
Step 10: Alternatively, if the alerts are consistently dismissible as false positives and there's no clear pattern, it might be worth turning them off temporarily and reassessing later.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In this scenario, deciding whether to tune or turn off an alert involves a thorough analysis of both its behavior and potential impacts. Tuning the alert is often preferred as it keeps the monitoring system effective while reducing noise. However, if tuning isn't possible due to complexity or resource constraints, turning off alerts should be done with careful consideration of their impact.

#### Common mistakes
include:

Ignoring the alert completely without understanding why it's a false positive.
Failing to gather sufficient data before making a decision on tuning vs. turning off.
Over-tuning which can lead to missing real issues due to overly strict thresholds.

#### Follow-ups
could involve discussing the specific metrics and thresholds being used, or exploring more advanced anomaly detection techniques such as machine learning models that Datadog supports.

**Comparison:**

```
2018-03-09
2018-03-11
2018-03-13
2018-03-15
2018-03-17
2018-03-19
2018-03-21
complete
effortYears
2018-03-15T14
complete
Data Collection
Analysis and Tuning
Implementation
Adjusting Alert Thresholds for 'example-service'
```

**Code:**

```python
# Example code for threshold tuning in Datadog API using Python
import requests

def adjust_alert_threshold(service):
    # Adjust alert threshold based on historical data analysis
    url = f"https://api.datadoghq.com/api/v1/service/{service}/alert/threshold"
    headers = {
        "Content-Type": "application/json",
        "DD-API-KEY": "YOUR_API_KEY",
        "DD-APPLICATION-KEY": "YOUR_APPLICATION_KEY"
    }

    # Example payload to adjust the threshold
    payload = {
        "value": 95,  # New threshold value based on analysis
        "service_id": service.id
    }

    response = requests.patch(url, headers=headers, json=payload)
    if response.status_code == 200:
        print("Threshold adjusted successfully.")
    else:
        print(f"Failed to adjust threshold: {response.text}")

adjust_alert_threshold(service)
```

**Example input / output:**

| Input | Output |
|-------|--------|
| Service: example-service, Metrics: request.latency (threshold 90) | Threshold adjusted to 95 for the 'request.latency' metric in 'example-service'. |

**Data:**

```json
{
  "service": {
    "id": 12345,
    "name": "example-service"
  },
  "metrics": [
    {
      "name": "request.latency",
      "thresholds": {
        "value": 90,
        "window": 60
      }
    }
  ]
}
```

**Flow:**

1. Review Recent Logs andMetrics
2. Tune or Refine the Alert
3. Evaluate Impact of FalsePositives
4. Turn Off Alerts Temporarily
5. Reassess Later
6. Consider Alternative Actions

