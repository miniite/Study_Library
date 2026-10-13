---
type: "practice"
difficulty: "Intermediate"
---

# How does Datadog pricing work at a high level for hosts custom metrics and log events?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Metrics vs Logs Monitoring](<../../../../TOOLS/Datadog/Datadog_Platform/Metrics_vs_Logs_Monitoring.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: Datadog pricing for hosts, custom metrics, and log events is based on the total volume of data processed across your infrastructure. The cost varies depending on the amount of data collected and retained, with different plans offering varying levels of support.

#### Triaging the Question
The interviewer is asking about the high-level pricing structure of Datadog for monitoring hosts, collecting custom metrics, and handling log events. They want to understand how these features are priced together in Datadog's offerings. We need to consider both the data volume aspect and the tiered pricing model.

#### Assumptions

We're discussing a basic understanding of Datadog's hosting services.
Custom metrics refer to user-defined performance or operational metrics.
Log events include application logs, system logs, etc., that are ingested by Datadog.

#### First-principles Solution
At its core, Datadog pricing is based on the total volume of data ingested and the amount of time you retain this data. This includes:

Metered Data Usage: Datadog bills per million bytes (MB) of raw log data or per 500K custom metrics collected in a month.
Retention Policies: Different plans allow different retention periods for your data, affecting the cost if you choose to retain data for longer.

Here’s a step-by-step breakdown:

Step 1: Understanding Metered Data Usage
Custom Metrics: Each custom metric is billed as 0.5 million bytes (MB) per month.
Logs: Datadog bills based on the number of bytes ingested and stored:
1 MB of data costs $0.23.
Step 2: Retention Policies
Basic Plan: 7 days of retention, limited to a certain amount of data.
Standard Plan: 30 days of retention, with more flexibility in data storage.
Enterprise Plans: Customizable retention periods and higher storage limits, often offering additional features like integrations and support.
Step 3: Cost Calculation

To calculate the cost:

Determine the total amount of custom metrics and log bytes ingested.
Multiply by the respective costs (0.5 MB for each custom metric and $0.23 per MB for logs).
Add any additional features or plans you have chosen.
Step 4: Example Calculation

Suppose a user has:

1,000 custom metrics per month.
1 GB of log data ingested and retained for 30 days.

Cost Calculation:

Custom Metrics Cost: ( 1,000 \text{ custom metrics} \times 0.5 \text{ MB/metric} = 500 \text{ MB} )

Since each MB costs $0.23, the cost is ( 500 \times 0.23 = $115 ) per month.

Log Data Cost:

1 GB (1,024 MB) of data at $0.23/MB:
Total log cost: ( 1,024 \text{ MB} \times 0.23 = $235.52 )

Total Monthly Cost:

Basic plan might include a fixed cost for 7 days of retention.
For the Standard or Enterprise plans, these costs would be adjusted.

#### Closing Summary
and Common Mistakes/Follow-Ups:

Datadog pricing is designed to scale with your data volume needs. The key takeaway is understanding metered usage and retention policies. Frequent overage can lead to higher costs if you exceed the baseline provided by a plan. Follow-ups might involve discussing how Datadog’s plans accommodate growth or specific requirements like compliance or security.

**Comparison:**

```
Data Type	Cost per Unit
Custom Metrics	0.23 MB/metric
Log Data (Bytes)	0.23 MB
```

**Code:**

```python
# Example Python script for cost calculation (simplified)
def calculate_custom_metric_cost(custom_metrics):
    return custom_metrics * 0.5 * 0.23

def calculate_log_data_cost(log_bytes):
    return log_bytes * 0.23

custom_metrics = 1000
log_bytes = 1024 * 1024  # 1 GB in bytes
custom_metric_cost = calculate_custom_metric_cost(custom_metrics)
log_data_cost = calculate_log_data_cost(log_bytes)

total_cost = custom_metric_cost + log_data_cost
print(f"Total cost: ${total_cost:.2f}")
```

**Example input / output:**

```text
Input:
- Custom metrics: 1000
- Log bytes: 1 GB

Output:
- Cost for custom metrics: $115
- Cost for log data: $235.52
- Total cost: $350.52 per month
```

**Data:**

```json
{
  "custom_metrics": 1000,
  "log_bytes": 1073741824,  # 1 GB in bytes
  "custom_metric_cost_per_mb": 0.23 / 500 * 0.5,
  "log_cost_per_mb": 0.23
}
```

**Flow:**

1. Yes
2. Yes
3. No
4. Start
5. Is there a custom metric?
6. Calculate cost as 0.5 MB
7. Is log data ingested?
8. Calculate cost as $0.23/MB
9. Add both costs for totalmonthly cost
10. No additional costs
11. End

