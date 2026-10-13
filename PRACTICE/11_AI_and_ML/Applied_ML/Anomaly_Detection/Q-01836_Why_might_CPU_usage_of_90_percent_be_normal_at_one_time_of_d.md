---
type: "practice"
difficulty: "Basic"
---

# Why might CPU usage of 90 percent be normal at one time of day but anomalous at another?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Seasonality and Trend Handling](<../../../../11_AI_and_ML/Applied_ML/Anomaly_Detection/Seasonality_and_Trend_Handling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When considering why CPU usage of 90% might be considered normal during certain times and anomalous during others, we're delving into the concept of context-dependent anomaly detection in time series data. This is a fundamental idea in AIOps where the behavior at different points in time needs to be understood within its temporal context.

To start, let's reframe the question: Anomaly Detection (AD) systems aim to identify patterns or behaviors that deviate significantly from historical norms. For CPU usage, what’s normal can change with the day of the week, time of day, user activity, and other factors. The key is understanding these temporal contexts.

We assume we have access to a dataset containing historical CPU usage data over several days or weeks. We are interested in pinpointing when 90% CPU utilization would be flagged as anomalous compared to its normal state.

**Comparison:**

```
Time of Day	Weekdays (Mon-Fri)	Weekends & Holidays
8 AM - 5 PM	> 80% anomalies	> 75% anomalies

This table shows the different thresholds for anomaly detection based on time.
```

**Code:**

```python
# Example of basic time series data structure for CPU usage
cpu_usage_data = [
    {'timestamp': '2023-10-05T08:00:00', 'usage': 90},
    # ... more entries
]
```

**Example input / output:**

```text
For example, at 8 AM on a typical weekday (Monday to Friday), having 90% CPU utilization might be considered normal. However, during the same time slot on weekends or holidays, such high CPU usage could indicate an anomaly.

Here’s how we can reason through this:

1. **Historical Data Context**: We would first build a baseline of normal behavior by analyzing historical data.
2. **Temporal Patterns**: Understanding daily and weekly patterns helps in setting thresholds that change based on time.
3. **Real-Time Monitoring**: During real-time monitoring, these thresholds are applied to detect deviations.

To illustrate with a simple example:
- On weekdays (Monday-Friday) from 9 AM - 5 PM, the threshold for anomaly detection might be set such that any usage over 80% triggers an alert.
- On weekends or holidays, this threshold could drop to 75%, making 90% more likely to trigger an alert.

This context-dependency is crucial in AIOps, as systems need to adapt to varying operational conditions and user behaviors.
```

**Data:**

```text
The `cpu_usage_data` list contains timestamps and corresponding CPU usage percentages.
```

**Flow:**

1. Receive CPU usage data
2. Collect historical data
3. Identify temporal patterns
4. Apply dynamic thresholds
5. Detect anomalies
6. Take action
7. This flowchart illustrates the steps from receiving data to detecting and acting on anomalies.

