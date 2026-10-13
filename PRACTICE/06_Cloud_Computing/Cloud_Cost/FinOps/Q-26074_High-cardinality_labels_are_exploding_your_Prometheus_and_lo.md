---
type: "practice"
difficulty: "Deep"
---

# High-cardinality labels are exploding your Prometheus and log costs. How do you find the offending labels and what drop or sampling rules do you apply without losing incident-debug signal?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Cost Anomaly Detection](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Cost_Anomaly_Detection.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address high-cardinality labels in Prometheus and log monitoring, we need to identify which labels are causing excessive cost spikes and then apply appropriate drop or sampling rules while ensuring that critical incident debugging signals remain intact. This involves a multi-step process from data analysis to rule optimization.

Firstly, let's understand the core issue: high cardinality leads to an exponential growth in time series data, resulting in higher storage costs. We need to identify the labels contributing most significantly to this problem and then apply cost-effective strategies like sampling or dropping less critical data.

#### Triage
and Assumptions
Assume we are dealing with a large-scale cloud environment where Prometheus is used for monitoring and logging.
High cardinality labels refer to those that have many unique values, leading to an explosion in the number of time series.
The goal is to maintain critical incident debugging signals without significantly impacting cost.

#### First-Principles Solution
Data Analysis: Use Prometheus query tools like time_series_explorer or Grafana to analyze which labels are causing the most data growth.
Impact Assessment: Identify critical services and labels that must retain full detail for effective incident debugging.
Rule Application:
Sampling Rules: Apply sampling rules selectively based on label values, retaining more detailed data for high-impact operations or specific environments.
Drop Rules: Gradually drop less critical labels in non-critical environments to reduce storage costs.
Step-by-Step Solution

Identify High-Cardinality Labels

SH
# Example query using Prometheus Query Explorer
time_series_explorer {
  label: "environment"
  aggregation: "count"
}

This helps in identifying labels with a large number of unique values.

Assess Criticality and Impact

Critical Services: Identify services that need full detail for debugging.
Non-Critical Services: For non-critical services, we can apply sampling or drop rules selectively.

Apply Sampling Rules

YAML
# Example sampling rule
- job_name: "prometheus"
  honor_labels: true
  interval: 15s
  scheme: http
  static_configs:
    - targets:
      labels:
        environment: !Sampling
          probability: 0.9 # Retain 90% of the data, drop rest

# Define sampling based on label values
rules:
  - rtype: "rate"
    expr: "up{environment=\"staging\"}"
    record: "up_rate_staging"
    interval: 15s
    target_label: "sampling"
    target_value: "true" # True for 90% retention, false otherwise

Apply Drop Rules

YAML
# Example drop rule
- job_name: "prometheus"
  honor_labels: true
  interval: 15s
  scheme: http
  static_configs:
    - targets:
      labels:
        environment: !Drop
          if: "environment != 'production'" # Drop all non-production environments

rules:
  - rtype: "rate"
    expr: "up{environment=\"staging\"}"
    record: "up_rate_staging"
    interval: 15s
    target_label: "sampling"
    target_value: "false" # False for dropping data

#### Closing Summary
and Common Mistakes/Follow-Ups
Mistake: Dropping too many critical labels without proper assessment.

#### Follow-Up
Regularly review the effectiveness of sampling and drop rules to ensure they are balanced between cost savings and debugging needs.

By following these steps, we can effectively manage high-cardinality labels in Prometheus and log monitoring while maintaining essential incident debugging capabilities. The key is a balanced approach that involves thorough data analysis and strategic rule application.

**Comparison:**

| Environment | Sampling Probability | Drop Condition |
|-------------|----------------------|---------------|
| Production  | 100%                 | N/A           |
| Staging     | 90%                  | True          |
| Development | 80%                  | True          |
| Test        | 50%                  | True          |
| Others      | 30%                  | True          |

**Code:**

```yaml
# Example of sampling and drop rules configuration
- job_name: "prometheus"
  honor_labels: true
  interval: 15s
  scheme: http
  static_configs:
    - targets:
      labels:
        environment: !Sampling
          probability: 0.9 # Retain 90% of the data, drop rest

rules:
  - rtype: "rate"
    expr: "up{environment=\"staging\"}"
    record: "up_rate_staging"
    interval: 15s
    target_label: "sampling"
    target_value: "true" # True for 90% retention, false otherwise

- job_name: "prometheus"
  honor_labels: true
  interval: 15s
  scheme: http
  static_configs:
    - targets:
      labels:
        environment: !Drop
          if: "environment != 'production'" # Drop all non-production environments

rules:
  - rtype: "rate"
    expr: "up{environment=\"staging\"}"
    record: "up_rate_staging"
    interval: 15s
    target_label: "sampling"
    target_value: "false" # False for dropping data
```

**Example input / output:**

```text
Input:
- Prometheus query explorer results indicating high-cardinality labels.
- Metrics data with different environment labels.

Output:
- Identified critical services and non-critical services.
- Applied sampling rules to retain detailed data for critical environments.
- Dropped less critical label values in non-production environments to reduce storage costs.

Example Rule Application:
Sampling rule applied: 90% retention of 'staging' environment data, with full detail maintained.
Drop rule applied: Non-'production' environment labels dropped to save on costs while retaining important metrics.
```

**Data:**

```json
{
  "prometheus": {
    "job_name": "prometheus",
    "honor_labels": true,
    "interval": "15s",
    "scheme": "http",
    "static_configs": [
      {
        "targets": [],
        "labels": {
          "environment": "!Sampling"
        },
        "rules": [
          {
            "rtype": "rate",
            "expr": "up{environment=\"staging\"}",
            "record": "up_rate_staging",
            "interval": "15s",
            "target_label": "sampling",
            "target_value": "true" // True for 90% retention, false otherwise
          }
        ]
      },
      {
        "targets": [],
        "labels": {
          "environment": "!Drop"
        },
        "rules": [
          {
            "rtype": "rate",
            "expr": "up{environment=\"staging\"}",
            "record": "up_rate_staging",
            "interval": "15s",
            "target_label": "sampling",
            "target_value": "false" // False for dropping data
          }
        ]
      }
    ]
  }
}
```

**Flow:**

1. Yes
2. No
3. Identify High-CardinalityLabels
4. Are labels critical?
5. Apply Sampling Rules
6. Apply Drop Rules
7. Monitor and Optimize
8. Monitor and Optimize

