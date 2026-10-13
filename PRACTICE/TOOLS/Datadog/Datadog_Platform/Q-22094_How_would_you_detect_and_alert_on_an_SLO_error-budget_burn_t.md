---
type: "practice"
difficulty: "Deep"
---

# How would you detect and alert on an SLO error-budget burn that only shows up at the tail of the latency distribution?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Anomaly and Forecast Monitors](<../../../../TOOLS/Datadog/Datadog_Platform/Anomaly_and_Forecast_Monitors.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of detecting and alerting on an SLO (Service Level Objective) error-budget burn specifically in the tail of the latency distribution, we need to leverage Datadog's advanced analytics capabilities and implement a custom monitoring solution that focuses on outlier detection. This approach ensures that we are sensitive to anomalies that occur infrequently but have significant impact.

#### Triage

The core question revolves around identifying when the tail latency (e.g., 95th or 99th percentile) starts to degrade, even if the overall SLO is still within acceptable bounds. Datadog provides rich monitoring tools and alerting capabilities that can be tailored to detect such anomalies.

#### First-Principles Solution
Define Custom Metrics: Use Datadog's custom metrics feature to track tail latencies.
Set Up Alerting Rules: Create alert rules based on these custom metrics to trigger when the 95th or 99th percentile latency exceeds a threshold for an extended period.
Outlier Detection: Implement an anomaly detection algorithm within Datadog to flag periods where the tail latencies consistently exceed normal ranges, even if they are within SLO limits.
Detailed Solution

Custom Metrics Setup:

Define a custom metric in Datadog for 95th and 99th percentile latency.
PLAINTEXT
metrics: [service_name].latency.pct[95], [service_name].latency.pct[99]

Alert Rules Configuration:

Create alert rules that trigger if these tail latencies exceed a threshold (e.g., 10 seconds) for more than a certain period (e.g., 5 minutes).
PLAINTEXT
resource: my_service
metric: [service_name].latency.pct[95]
filter by: {environment: production}
alert when: >10s over 5m

Anomaly Detection:

Use Datadog's anomaly detection feature to set up a baseline and detect deviations.
PLAINTEXT
resource: my_service
metric: [service_name].latency.pct[95]
filter by: {environment: production}
alert when: deviates from baseline for 10m

#### Closing Summary
and Common Mistakes/Follow-Ups

By setting up these custom metrics, alert rules, and anomaly detection, we ensure that even if the overall SLO is not breached, significant issues in tail latencies can be detected early. This approach provides a more granular view of service performance and helps in maintaining high availability and user satisfaction.

#### Common Mistakes

Not defining clear threshold values for alerting, which could lead to false positives or negatives.
Failing to set up proper baseline detection periods, leading to delayed alerts.
Ignoring the periodicity of checks, resulting in missed anomalies during off-peak times.

For follow-ups, one might explore integrating this solution with Datadog's APM (Application Performance Monitoring) tools to get deeper insights into the root causes of these latency issues. Additionally, exploring machine learning models for more sophisticated anomaly detection could further enhance the solution.

**Comparison:**

| Custom Metrics | Description |
|----------------|-------------|
| [service_name].latency.pct[95] | Tracks the 95th percentile latency of service_name. |
| [service_name].latency.pct[99] | Tracks the 99th percentile latency of service_name. |
| Alert Rules | Description |
|-------------|-------------|
| High_95th_Percentile_Latency | Triggers when 95th percentile latency exceeds 10s for more than 5 minutes in production environment. |

**Code:**

```plaintext
# Example alert rule configuration in YAML
resources:
  - name: my_service
    alerts:
      - name: High_95th_Percentile_Latency
        query: [service_name].latency.pct[95]
        filters:
          - environment: production
        conditions:
          - value: >10s
            over: 5m
```

**Example input / output:**

```text
# Example input and output for custom metrics setup
Input:
- Service Name: MyApp
Output:
- Metrics Created:
  - [MyApp].latency.pct[95] (distribution)
  - [MyApp].latency.pct[99] (distribution)

# Alert rule creation example
Alert Rule:
- Resource: MyApp
- Metric: [MyApp].latency.pct[95]
- Filter by: environment: production
- Condition: >10s over 5m
```

**Data:**

```plaintext
# Example custom metrics setup in Datadog UI or API
custom_metrics:
  - name: [service_name].latency.pct[95]
    type: distribution
  - name: [service_name].latency.pct[99]
    type: distribution
```

**Flow:**

1. Define Custom Metrics
2. Set Up Alert Rules
3. Configure Anomaly Detection
4. Monitor and Trigger Alerts

