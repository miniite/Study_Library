---
type: "practice"
difficulty: "Deep"
---

# Your streaming platform's consumer lag spiked during an incident and the auto-scaler spun up dozens of extra Flink task managers that never scaled back, quietly becoming permanent baseline cost. What signal distinguishes emergency stream-processing capacity that should sunset from capacity that legitimately became steady-state, and how do you enforce the paydown?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Usage-based Cost Surprises](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Usage-based_Cost_Surprises.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of distinguishing between emergency stream-processing capacity that should sunset versus capacity that has become a steady state, we need to understand the dynamics of both situations. The core challenge is ensuring that cost governance aligns with actual processing needs while minimizing unnecessary expenditure.

In an emergency scenario, capacity is rapidly increased due to unexpected spikes in data volume or processing load. This situation requires immediate action but also risks introducing permanent changes if not properly managed. On the other hand, steady-state capacity reflects normal operating conditions where resource allocation has stabilized and can be optimized for cost efficiency.

To distinguish these scenarios and enforce paydowns effectively, we need a robust monitoring and governance framework that leverages historical data, anomaly detection, and automated policies.

**Comparison:**

| Timestamp       | Task Manager Count |
|-----------------|--------------------|
| 2023-10-01T12:00Z | 5                  |
| 2023-10-01T12:10Z | 6                  |
| 2023-10-01T12:20Z | 8                  |
| Threshold       | Anomaly Detected? |
|-----------------|--------------------|
| 3               | Yes                |

**Code:**

```python
# Example function to detect anomalies in streaming metrics
def is_anomaly(metric_data):
    threshold = 3  # Define the standard deviation threshold
    mean, std_dev = calculate_mean_std_dev(metric_data)
    return abs(metric_data - mean) > threshold * std_dev

def calculate_mean_std_dev(data_points):
    mean = sum(data_points) / len(data_points)
    variance = sum((x - mean) ** 2 for x in data_points) / len(data_points)
    std_dev = sqrt(variance)
    return mean, std_dev
```

**Example input / output:**

```text
| Timestamp       | Task Manager Count |
|-----------------|--------------------|
| 2023-10-01T12:00Z | 5                  |
| 2023-10-01T12:10Z | 6                  |
| 2023-10-01T12:20Z | 8                  |

Based on the sample data, an anomaly is detected as the count significantly deviates from its mean and standard deviation.
```

**Data:**

```json
{
  "stream_metrics": {
    "task_managers": [
      {"timestamp": "2023-10-01T12:00Z", "count": 5},
      {"timestamp": "2023-10-01T12:10Z", "count": 6},
      {"timestamp": "2023-10-01T12:20Z", "count": 8}
    ]
  }
}
```

**Flow:**

1. Yes
2. No
3. Monitor Streaming Metrics
4. Is Anomaly Detected?
5. Maintain Emergency Capacity
6. Analyze Steady-State Data
7. Adjust Capacity to Steady State
8. Enforce Cost GovernancePolicies
9. Monitor and Rebalance

