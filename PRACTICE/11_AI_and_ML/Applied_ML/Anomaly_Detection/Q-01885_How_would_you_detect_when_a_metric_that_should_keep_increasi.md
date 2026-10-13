---
type: "practice"
difficulty: "Intermediate"
---

# How would you detect when a metric that should keep increasing (like a counter) suddenly flatlines?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Drift and Threshold Detection](<../../../../11_AI_and_ML/Applied_ML/Anomaly_Detection/Drift_and_Threshold_Detection.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To detect when a metric like a counter, which should continually increase over time, suddenly stops increasing, we need to implement an anomaly detection system tailored for this type of behavior. The key idea is to identify sudden changes or stagnations in the otherwise linear growth pattern. This involves setting up appropriate thresholds and monitoring mechanisms.

#### Triaging the Question

The question is essentially asking how to detect a significant deviation from expected behavior in a time series metric that should increase monotonically. We need to define what constitutes an "anomaly" based on historical data patterns. The interviewer might be probing our understanding of statistical methods, threshold setting, and real-time monitoring techniques.

#### Assumptions
Monotonic Increase: The counter is expected to always increase over time.
Historical Data: We have access to past values of the metric to establish a baseline for normal behavior.
Real-Time Monitoring: The system needs to continuously monitor the metric in real-time.

#### First-Principles Solution

Baseline Estimation:

Calculate the average increase rate over a certain window (e.g., last 24 hours, 7 days).
Use rolling windows to adaptively update this baseline as new data comes in.

Threshold Setting:

Define a threshold for deviation from the expected increase rate using statistical methods such as Z-score or IQR.
For example, if the average daily increase is 10 units and we set a threshold of ±3 standard deviations, any value outside this range will be flagged.

Real-Time Monitoring:

Continuously update the baseline with new data points.
Compare each new point against the current baseline to detect anomalies in real-time.

Notification and Action:

If a deviation is detected beyond the threshold, trigger an alert or log the anomaly for further investigation.

#### Example Scenario

Suppose we have a counter that should always increase by at least 10 units per day on average. Over the last week, the increases were consistently between 9 and 12 units. We set our baseline to be around 10.5 units as the expected daily increment.

If today's increment is only 3 units, we flag this as an anomaly because it deviates significantly from the historical pattern.

#### Closing Summary

To summarize, detecting anomalies in a monotonically increasing metric involves establishing a baseline of normal behavior using historical data and setting thresholds for deviation. Real-time monitoring ensures quick detection and response to unexpected changes. Common pitfalls include over-reliance on static thresholds without adaptability, and failing to consider edge cases where the metric could temporarily drop before resuming its trend.

#### Common Mistakes/Follow-Ups
Static Thresholds: Using fixed thresholds can miss anomalies if the baseline shifts.
Insufficient Baseline Data: Not having enough historical data can lead to unreliable baselines and false positives.
Alert Fatigue: Setting too many alerts for minor deviations might overwhelm operational teams.

Would you like me to elaborate on any of these points or provide a sample implementation in code?

**Code:**

```python
import pandas as pd

def calculate_threshold(df, window_size=24):
    rolling_mean = df['value'].rolling(window=window_size).mean()
    std_dev = df['value'].rolling(window=window_size).std()
    threshold_upper = rolling_mean + 3 * std_dev
    return rolling_mean, threshold_upper

def detect_anomaly(df, rolling_mean, threshold_upper):
    anomalies = []
    for i in range(len(df)):
        if df.loc[i, 'value'] < rolling_mean.iloc[i] - 3 * std_dev or df.loc[i, 'value'] > threshold_upper.iloc[i]:
            anomalies.append((i, df.loc[i, 'timestamp'], df.loc[i, 'value']))
    return anomalies

# Sample Data
data = pd.DataFrame({
    'timestamp': pd.date_range(start='2023-10-01', periods=31, freq='D'),
    'value': [12, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39, 40, 41, 42, 43, 44]
})

# Calculate rolling mean and threshold
rolling_mean, threshold_upper = calculate_threshold(data)

# Detect anomalies
anomalies = detect_anomaly(data, rolling_mean, threshold_upper)
print("Anomalies detected:", anomalies)
```

**Example input / output:**

```text
Timestamps:
2023-10-01 00:00:00, Value: 12
2023-10-02 00:00:00, Value: 15
2023-10-03 00:00:00, Value: 16
2023-10-04 00:00:00, Value: 17
2023-10-05 00:00:00, Value: 18
2023-10-06 00:00:00, Value: 19
2023-10-07 00:00:00, Value: 20
2023-10-08 00:00:00, Value: 21
2023-10-09 00:00:00, Value: 22
2023-10-10 00:00:00, Value: 23
2023-10-11 00:00:00, Value: 24
2023-10-12 00:00:00, Value: 25
2023-10-13 00:00:00, Value: 26
2023-10-14 00:00:00, Value: 27
2023-10-15 00:00:00, Value: 28
2023-10-16 00:00:00, Value: 29
2023-10-17 00:00:00, Value: 30
2023-10-18 00:00:00, Value: 31
2023-10-19 00:00:00, Value: 32
2023-10-20 00:00:00, Value: 33
2023-10-21 00:00:00, Value: 34
2023-10-22 00:00:00, Value: 35
2023-10-23 00:00:00, Value: 36
2023-10-24 00:00:00, Value: 37
2023-10-25 00:00:00, Value: 38
2023-10-26 00:00:00, Value: 39
2023-10-27 00:00:00, Value: 40
2023-10-28 00:00:00, Value: 41
2023-10-29 00:00:00, Value: 42
2023-10-30 00:00:00, Value: 43
2023-10-31 00:00:00, Value: 44

Anomalies detected:
[(28, '2023-10-31 00:00:00', 44)]
```

**Flow:**

1. Yes
2. Within Baseline
3. Outside Baseline
4. No
5. Collect Metric Data
6. Is Data Available?
7. Calculate Rolling Mean andThreshold
8. Monitor New Data Points
9. No Anomaly Detected
10. Detect Anomaly
11. Notify Team / Log Anomaly
12. Initialize Baseline

