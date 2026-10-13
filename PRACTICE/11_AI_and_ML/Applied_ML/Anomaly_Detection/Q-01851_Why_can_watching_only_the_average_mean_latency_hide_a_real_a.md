---
type: "practice"
difficulty: "Intermediate"
---

# Why can watching only the average (mean) latency hide a real anomaly that percentiles reveal?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Smoothing and Sensitivity Trade-offs](<../../../../11_AI_and_ML/Applied_ML/Anomaly_Detection/Smoothing_and_Sensitivity_Trade-offs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand why simply monitoring the mean latency might miss anomalies, we need to delve into how these two statistical measures behave differently when applied to time series data, especially in AIOps scenarios where latency can vary significantly across different percentiles.

#### Triaging the Question

The question is probing our understanding of anomaly detection techniques and the nuances between using mean (average) latency versus percentiles. Mean latency gives us an idea of the central tendency but doesn't account for extreme values or outliers, which are crucial in many AIOps applications where anomalies often manifest as sudden spikes.

#### First-Principles Solution

Let’s start with a simple scenario: consider a network service whose response time (latency) is normally around 50ms. However, occasionally the latency can spike to several seconds due to network congestion or other issues. Here's how mean and percentiles behave:

Mean Latency: The average of all latencies over a period.
Percentiles (e.g., 95th percentile): A threshold that separates the fastest 5% from the rest.

In our example:

Suppose in a given minute, most requests have a latency around 50ms, but one request hits a spike of 10 seconds due to network issues.
The mean latency for this minute could still be close to 50ms if there are enough other latencies near 50ms to offset the outlier.
Detailed Reasoning
Case Study: Latency Distribution

#### Cons
ider a dataset with the following latencies in milliseconds:

CODE
[40, 42, 43, 47, 10000]
Mean: (\frac{40 + 42 + 43 + 47 + 10000}{5} = 2060.8) ms
95th Percentile (assuming no other data points): 10000 ms

Clearly, the mean is heavily influenced by the outlier, which might not be representative of normal operations.

Real-world Example

In a large-scale web application, user experience can degrade significantly due to rare but critical latency spikes. If you only monitor the mean latency, these anomalies might go unnoticed as long as other latencies are within acceptable bounds.

#### Closing Summary
and Common Mistakes

Monitoring only the mean latency can indeed miss important anomalies because it averages out extreme values. Percentiles provide a better picture by highlighting outliers and ensuring that critical issues (like sudden spikes) don't get lost in the noise of average values.

#### Common mistakes

Over-relying on mean without considering percentiles: This can lead to underestimating real issues.
Not setting up proper threshold-based alerts: Relying solely on statistical measures without dynamic alert thresholds based on historical data.
Ignoring the business context: What might seem like a small average change could have significant impacts, especially in high-stakes operations.

#### Follow-up
Questions
How would you set up an anomaly detection system that combines both mean and percentiles?
Can you explain how to dynamically adjust thresholds based on historical data in your AIOps environment?

**Code:**

```python
import numpy as np

# Sample dataset of latencies
latencies = [40, 42, 43, 47, 10000]

# Calculate mean and 95th percentile
mean_latency = np.mean(latencies)
percentile_95 = np.percentile(latencies, 95)

print(f"Mean Latency: {mean_latency} ms")
print(f"95th Percentile: {percentile_95} ms")
```

**Example input / output:**

```

Input:

PYTHON
Copy
latencies = [40, 42, 43, 47, 10000]

Output:

PLAINTEXT
Copy
Mean Latency: 2060.8 ms
95th Percentile: 10000.0 ms
```

