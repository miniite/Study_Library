---
type: "practice"
difficulty: "Deep"
---

# How would you architect governance so a data-quality regression in an upstream service's metrics is caught before it silently degrades the accuracy of anomaly detection?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Certifying Data Sources](<../../../../05_Data_Governance_and_Security/Governance/Data_Strategy/Certifying_Data_Sources.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we need to ensure that any data quality issues in an upstream service are detected and addressed before they propagate down the pipeline and impact downstream systems like anomaly detection. The key is to implement a robust governance strategy that includes real-time monitoring, automated checks, and proactive alerting mechanisms.

The question is probing into how you would design a system for continuous data validation and monitoring, with a focus on ensuring data integrity at each step of the data flow. This involves several layers: data validation rules, anomaly detection thresholds, and effective logging and alerting systems.

#### Assumptions

The upstream service produces time-series metrics.
We need to validate these metrics against known standards or business rules.
There's a downstream system (anomaly detection) that relies on the accuracy of the upstream metrics.

Step-by-step reasoning from first principles:

Define Data Quality Checks: Identify and define the critical data quality checks for the time-series metrics. These could include:

Format correctness (e.g., data type, date formats).
Range validation (e.g., minimum and maximum values).

#### Cons
istency of measurements across different instances or timestamps.
Presence of outliers.

Implement Real-Time Monitoring: Set up real-time monitoring for the upstream service using tools like Prometheus, Grafana, or custom-built dashboards. These tools can continuously validate incoming data against predefined rules and alert on any deviations.

Integrate with Anomaly Detection System: Ensure that the anomaly detection system is aware of these data quality issues. This can be done by integrating a health check endpoint in the upstream service that reports its status. The anomaly detection system can then use this information to adjust its thresholds or even halt processing if necessary.

Automated Alerts and Notifications: Configure automated alerts for any failed validation checks. Use tools like PagerDuty, Slack, or email notifications to alert relevant stakeholders (e.g., data engineers, developers) as soon as an issue is detected.

Logging and Retention: Implement logging for all validation activities. Ensure that logs are retained long enough to support post-mortem analysis if issues arise.

Feedback Mechanism: Develop a feedback loop where the anomaly detection system can provide insights back to the upstream service about common patterns of errors or anomalies. This can help in refining data quality checks and improving overall data integrity.

#### Closing summary
The solution involves a combination of real-time monitoring, automated validation rules, alerts, and a robust logging mechanism. By setting up these layers, we ensure that any issues with data quality are detected early and addressed proactively, thereby maintaining the accuracy of downstream systems like anomaly detection.

#### Common mistakes/follow-ups

Failing to define clear data quality checks.
Overlooking the importance of real-time monitoring and automated alerts.
Not integrating feedback mechanisms from downstream systems back into upstream services.
Insufficient logging and retention policies.

**Comparison:**

```
Timestamp	Value	Status
2023-10-01T00:00:00Z	98.5	valid
2023-10-01T01:00:00Z	106.5	invalid
```

**Code:**

```python
# Example of a simple validation function in Python

def validate_metric(metric_value, lower_bound, upper_bound):
    if not isinstance(metric_value, (int, float)):
        return False, "Invalid data type"
    if metric_value < lower_bound or metric_value > upper_bound:
        return False, f"Value out of bounds: {metric_value}"
    return True, None

# Example usage
valid, msg = validate_metric(102, 95, 105)
if not valid:
    print(f"Validation failed: {msg}")
```

**Example input / output:**

```text
| Timestamp       | Value   | Status         |
|-----------------|---------|----------------|
| 2023-10-01T00:00:00Z | 98.5     | valid          |
| 2023-10-01T01:00:00Z | 106.5    | invalid        |

Alert generated for the timestamp 2023-10-01T01:00:00Z.
```

**Data:**

```json
{
  "metrics": [
    {
      "timestamp": "2023-10-01T00:00:00Z",
      "value": 98.5,
      "status": "valid"
    },
    {
      "timestamp": "2023-10-01T01:00:00Z",
      "value": 106.5,
      "status": "invalid"
    }
  ]
}
```

**Flow:**

1. Upstream Service
2. Validate Metrics
3. Log and Alert
4. Anomaly Detection System

