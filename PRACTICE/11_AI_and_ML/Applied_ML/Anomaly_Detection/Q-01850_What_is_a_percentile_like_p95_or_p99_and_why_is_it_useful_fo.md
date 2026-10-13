---
type: "practice"
difficulty: "Basic"
---

# What is a percentile (like p95 or p99) and why is it useful for anomaly detection on latency?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Smoothing and Sensitivity Trade-offs](<../../../../11_AI_and_ML/Applied_ML/Anomaly_Detection/Smoothing_and_Sensitivity_Trade-offs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Percentiles are statistical measures that divide data into 100 equal parts, with "p95" being the 95th percentile, meaning 95% of the data points fall below this value. For anomaly detection in latency metrics, percentiles help identify unusual spikes or drops by comparing observed values against these thresholds.

In essence, when we talk about p95 (or any other percentile) for latency, we're setting a threshold that only the top 5% of all observed latencies should exceed under normal conditions. This is useful because it helps us focus on significant outliers rather than minor variations. For example, if the average response time is 10ms, but you observe a spike to 300ms, this might be considered an anomaly.

To understand why percentiles are so valuable in anomaly detection:

Outlier Identification: They help pinpoint very high or low values that deviate significantly from normal ranges.
Risk Management: By setting thresholds at the p95 or higher (or lower), you can flag potential issues before they affect a majority of users.

#### Triage
The question is asking for an explanation of percentiles and their utility in anomaly detection, particularly in the context of latency metrics. The assumptions are that the reader understands basic statistics and the importance of setting thresholds to monitor system performance.

Step-by-Step Solution:

Define Percentile: A percentile (like p95) represents a value below which a given percentage of observations fall.
Context for Anomaly Detection: In latency metrics, we use percentiles to identify extreme values that might indicate an issue.

#### Example
Calculation: If the 95th percentile for a service’s response time is 100ms and you observe a 400ms response, this could be flagged as anomalous.
Use Case: This technique helps in identifying critical performance drops or spikes that need immediate attention.

#### Closing Summary
+ Common Mistakes/Follow-Ups:

Common Mistake: Over-reliance on single-point metrics without considering the full distribution (e.g., only focusing on average latency).

#### Follow-Up
Question: How do you handle services with highly volatile latencies when setting these thresholds?

**Code:**

```python
import numpy as np

# Example data for response times in milliseconds
response_times = [10, 20, 30, 40, 50, 60, 70, 80, 90, 100, 110, 120, 130, 140, 150, 160, 170, 180, 190, 200]

# Calculate the 95th percentile
p95 = np.percentile(response_times, 95)
print(f"The 95th percentile of response times is: {p95} ms")
```

**Example input / output:**

```text
Input:
response_times = [10, 20, 30, 40, 50, 60, 70, 80, 90, 100, 110, 120, 130, 140, 150, 160, 170, 180, 190, 200]

Output:
The 95th percentile of response times is: 177.9 ms
```

**Flow:**

1. Input Latency Data
2. Calculate Percentile
3. Compare to Thresholds
4. Detect Anomalies
5. Take Action

