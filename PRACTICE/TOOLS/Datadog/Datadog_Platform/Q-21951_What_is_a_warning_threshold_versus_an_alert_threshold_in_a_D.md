---
type: "practice"
difficulty: "Basic"
---

# What is a warning threshold versus an alert threshold in a Datadog monitor?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Composite Monitors](<../../../../TOOLS/Datadog/Datadog_Platform/Composite_Monitors.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with monitoring tools like Datadog, it’s essential to understand how thresholds are used to detect and respond to anomalies or issues in your system. In a Datadog monitor, warning thresholds and alert thresholds serve different purposes but both are crucial for effective real-time monitoring.

#### Triaging the Question

The question is asking you to differentiate between warning thresholds and alert thresholds within the context of a Datadog monitor. The interviewer might be probing your understanding of how these thresholds function in practice and their implications on system health checks and alerts.

From First Principles to Solution

In Datadog, monitoring involves setting up rules to detect specific conditions or behaviors in your systems. These are categorized into two main types: warning and alert thresholds.

Warning Thresholds
Definition: Warning thresholds are set levels that trigger a notification when exceeded but do not immediately result in an alert.
Purpose: They serve as early warnings to help you proactively address potential issues before they become critical.

#### Example Scenario
Suppose you have a metric that represents the average response time of your application. Setting a warning threshold at 50ms means you will be notified when this value is consistently above 50ms, allowing you to investigate and potentially resolve the issue early on.
Alert Thresholds
Definition: Alert thresholds are the critical levels at which an alert is triggered.
Purpose: Alerts are designed to immediately notify relevant stakeholders about significant issues that require immediate attention.

#### Example Scenario
Continuing with the response time metric, setting an alert threshold at 100ms means that if the average response time consistently exceeds this value, Datadog will send out an alert to designated users or teams.

#### Closing Summary
and Common Mistakes

In summary, warning thresholds are used for proactive monitoring and early detection of issues, while alert thresholds define the severity level at which actions must be taken immediately. Misunderstanding these roles can lead to unnecessary alerts or missed critical issues.

#### Common Mistakes/Follow-Ups
Overlapping Thresholds: A common mistake is setting warning and alert thresholds too close together. This can result in frequent, non-critical alerts that may clutter your notifications.
Ignoring Warning Signals: Failing to act on warnings can lead to critical issues being overlooked until they become alerts.
Improper Metric Selection: Choosing the wrong metric or threshold values can make both warning and alert thresholds less effective.
Triaging Common Follow-Ups

How do you determine appropriate threshold values?

Answer: Thresholds should be based on historical data analysis, understanding normal behavior, and business needs. For example, using percentiles of response times over time.

What happens if a metric consistently exceeds the warning threshold but never reaches the alert threshold?

Answer: This situation indicates that your thresholds might need adjustment. It’s important to review and tweak these values based on ongoing performance data.

**Comparison:**

```
Metric Name	Warning Threshold (ms)	Alert Threshold (ms)
service:web.response_time	50	100
```

**Code:**

```python
# Example code snippet for setting up Datadog monitors with thresholds (hypothetical)
import datadog

def setup_monitor(metric_name, warning_threshold, alert_threshold):
    tags = ['service:web']
    api_key = 'YOUR_API_KEY'
    app_key = 'YOUR_APP_KEY'

    # Setup the monitor
    monitor = {
        "name": f"{metric_name} Alert",
        "type": "gauge",
        "query": metric_name,
        "message": "Warning or alert for {#query}",
        "tags": tags,
        "options": {
            "evaluation_timeframe": {"from_now": "5m"},
            "thresholds": {
                "warning": warning_threshold,
                "alert": alert_threshold
            }
        },
        "notify_audit": True
    }

    # Create the monitor in Datadog
    datadog.api.Monitor.create(
        api_key=api_key,
        app_key=app_key,
        **monitor
    )
```

**Example input / output:**

```

Input:

PLAINTEXT
Copy
Metric Name: service:web.response_time
Warning Threshold: 50ms
Alert Threshold: 100ms

Output:

Warning Triggered: If the average response time is above 50ms for an extended period.
Alert Triggered: If the average response time consistently exceeds 100ms.
```

**Data:**

```json
{
  "service:web.response_time.warning_threshold": 50,
  "service:web.response_time.alert_threshold": 100
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. Monitor Setup
5. Is Response Time > 50ms?
6. Send Warning Notification
7. Review and Take Action
8. Is Response Time > 100ms?
9. Send Alert Notification
10. Notify Stakeholders, TakeImmediate Action

