---
type: "practice"
difficulty: "Intermediate"
---

# What is your approach to setting recovery thresholds and evaluation delays so a flapping service does not page on-call every few minutes?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Composite Monitors](<../../../../TOOLS/Datadog/Datadog_Platform/Composite_Monitors.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of a service flapping and causing frequent pagers, we need to understand that "flapping" refers to a situation where a monitored metric, like the availability or response time of a service, frequently crosses an alert threshold within a short period. The goal is to set recovery thresholds and evaluation delays so that alerts are only triggered when necessary, avoiding false positives.

In this context, we need to strike a balance between sensitivity (to catch real issues quickly) and noise reduction (to prevent frequent pagers). Let's walk through the steps to achieve this from first principles.

First, let's define what recovery thresholds and evaluation delays are:

Recovery Thresholds: These are secondary thresholds that determine whether an alert should be closed once it has been triggered. For example, if a service is currently down but then comes back up within 30 seconds, we might not want to trigger another page.
Evaluation Delays: These are the time periods during which alerts will not be evaluated or re-evaluated until they have passed. This helps in smoothing out noisy data.

Now, let's break it down step by step:

Understand the Flapping Behavior:

Identify the metrics that are causing flapping. For instance, HTTP response time might vary depending on network conditions.
Use Datadog to visualize these metrics over time and determine their patterns of fluctuation.

Set Initial Alert Thresholds:

Define the primary alert thresholds based on historical data. For example, if 95% of requests should return within 100ms, set an upper threshold at 300ms.
Ensure these thresholds are reasonable to avoid false positives but still sensitive enough to catch genuine issues.

Determine Recovery Thresholds:

Define secondary recovery thresholds that the metric must meet before closing an alert. For instance, if a request takes more than 15 seconds, set a recovery threshold of 60ms for it to be considered back to normal.
Use a combination of time and value thresholds here. A single large deviation might not trigger a recovery immediately but can be part of the criteria.

Implement Evaluation Delays:

Set evaluation delays based on the frequency and nature of flapping. For example, if flapping occurs every 10 seconds, an evaluation delay of at least 30-60 seconds is appropriate.

#### Cons
ider using sliding windows or moving averages to smooth out short-term noise.

Test and Iterate:

Apply these thresholds in a staging environment first to see how they behave with real data.
Monitor the results closely and adjust as needed. This might involve tweaking thresholds, changing evaluation delays, or even redefining what constitutes flapping behavior for your specific service.

In practice, using Datadog's custom alerting policies can help implement these settings effectively:

Datadog Alert Policies: Define alert conditions and actions in the Datadog UI.
Custom Logic: Use complex expressions to handle more nuanced alerts, such as evaluating over different time windows or applying threshold conditions.

By following this approach, we ensure that alerts are reliable and reduce noise from flapping metrics, leading to fewer unnecessary pagers for on-call engineers.

#### Common mistakes
include setting overly sensitive thresholds, not considering the nature of the metric (e.g., network latency vs. CPU usage), and failing to test thoroughly in a staging environment before rolling out changes.

**Comparison:**

| Metric | Primary Threshold (ms) | Recovery Threshold 1 (ms) | Recovery Threshold 2 (seconds) | Evaluation Delay (seconds) |
|--------|-----------------------|-------------------------|-------------------------------|---------------------------|
| http_response_time | 300                    | 60                       | 15                            | 30                        |

**Code:**

```python
# Example Datadog Alert Policy Setup in Python

def setup_alert_policy(metric_name, recovery_thresholds, evaluation_delay):
    # Define primary threshold
    primary_threshold = 300  # ms
    # Define recovery thresholds
    recovery_threshold1 = 60  # ms
    recovery_threshold2 = 15  # seconds

    # Define the alert policy in Datadog
    alert_policy = {
        "name": f"Alert on {metric_name} Flapping",
        "conditions": [
            {
                "query": f"{metric_name} > {primary_threshold}",
                "threshold_type": "sample",
                "alert_after": evaluation_delay,
                "recovery_thresholds": [recovery_threshold1, recovery_threshold2]
            }
        ],
        "actions": ["pagerduty"]
    }

    return alert_policy
```

**Example input / output:**

```
Input: setup_alert_policy("http_response_time", [60, 15], 30)
Output:
JSON
Copy
{
  "name": "Alert on http_response_time Flapping",
  "conditions": [
    {
      "query": "http_response_time > 300",
      "threshold_type": "sample",
      "alert_after": 30,
      "recovery_thresholds": [60, 15]
    }
  ],
  "actions": ["pagerduty"]
}
```

**Data:**

```json
{
  "metric_name": "http_response_time",
  "recovery_thresholds": [60, 15],
  "evaluation_delay": 30
}
```

**Flow:**

1. Define Metrics
2. Identify Flapping Patterns
3. Set Primary Thresholds
4. Define Recovery Thresholds
5. Implement Evaluation Delays
6. Test in Staging Environment
7. Monitor and Adjust

