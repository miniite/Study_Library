---
type: "practice"
difficulty: "Basic"
---

# Why is static threshold alerting often not enough for detecting anomalies in dynamic systems?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Dynamic vs Static Alerting](<../../../../11_AI_and_ML/Applied_ML/Anomaly_Detection/Dynamic_vs_Static_Alerting.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Static threshold alerting, while simple and easy to implement, can be insufficient for anomaly detection in dynamic systems because it does not account for the variability inherent in time series data. Let’s break down why this is a problem and explore better approaches.

#### Triage

The question probes into understanding the limitations of static thresholds in detecting anomalies within complex, evolving systems. Static thresholds are predefined and fixed values that trigger alerts when certain metrics exceed them. However, these thresholds do not adapt to changing conditions over time, which can lead to false negatives or false positives.

Step-by-Step Solution
1. Understanding the Problem

Static threshold alerting works by setting a fixed value above which an anomaly is detected. For instance, if you set a threshold for CPU usage at 80%, any time the CPU exceeds this value, an alert is triggered. This approach assumes that the metric remains relatively stable and does not change over time.

2. Identifying the Issues
False Negatives: If the system experiences sudden changes or seasonal fluctuations, static thresholds may fail to detect anomalies.
False Positives: In dynamic systems, normal but high values might trigger alerts, leading to unnecessary attention or intervention.
3. Better Approaches

To address these limitations, we can use more sophisticated methods such as:

Dynamic Thresholds: Adjusting the threshold based on historical data and recent trends.
Statistical Models: Utilizing statistical techniques like moving averages, exponential smoothing, or machine learning models to adaptively detect anomalies.
4. Example: Moving Average for Anomaly Detection

A common approach is to use a moving average (MA) to smooth out short-term fluctuations and highlight longer-term trends. For example, consider a time series of CPU usage data over the past hour:

Time	CPU Usage
t-5	78
t-4	79
t-3	80
t-2	81
t-1	82
t	83

Using a moving average over the last three points:

[ \text{MA}(t) = \frac{\text{CPU Usage}{t-3} + \text{CPU Usage}{t-2} + \text{CPU Usage}_t}{3} ]

If today's CPU usage is 85%, this value might be flagged as an anomaly because the moving average (79.67%) would likely be below it.

#### Closing Summary

In summary, static threshold alerting can lead to false positives and negatives in dynamic systems due to its inflexibility. Adopting more advanced techniques such as dynamic thresholds or statistical models can help detect anomalies more accurately by accounting for the variability inherent in time series data.

#### Common Mistakes
and Follow-Ups
Ignoring Seasonality: Forgetting that some systems have natural variations over time (e.g., daily cycles).
Overfitting Models: Using overly complex models that might not generalize well.
Not Updating Thresholds: Failing to adjust the threshold based on changes in system behavior.

Understanding these issues can help you choose the right approach for your specific use case, ensuring more robust and reliable anomaly detection in dynamic systems.

**Comparison:**

```
Time	CPU Usage	Moving Average
t-5	78	-
t-4	79	-
t-3	80	-
t-2	81	-
t-1	82	-
t	83	80.67
```

**Code:**

```python
# Example of a simple moving average calculation
def moving_average(data, window_size):
    return sum(data[-window_size:]) / window_size

cpu_usage_data = [78, 79, 80, 81, 82, 83]
window_size = 3
ma_value = moving_average(cpu_usage_data, window_size)
print(ma_value)  # Output: 80.67 (rounded to two decimal places for simplicity)
```

**Example input / output:**

```text
Input: CPU usage data over the past hour.
Output: Moving average value of 80.67 (assuming window size of 3).
```

**Data:**

| Time | CPU Usage |
|------|-----------|
| t-5  | 78        |
| t-4  | 79        |
| t-3  | 80        |
| t-2  | 81        |
| t-1  | 82        |
| t    | 83        |

**Flow:**

1. Yes
2. No
3. Collect CPU Usage Data
4. Is Time Series Length >=Window Size?
5. Moving Average = Sum of Lastn Points / n
6. Return Moving Average asThreshold
7. Wait for Next Sample andRepeat

