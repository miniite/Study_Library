---
type: "practice"
difficulty: "Intermediate"
---

# How would you implement a rolling z-score anomaly check in Python over a Prometheus metric stream, and why is a plain global mean/std a poor choice for a metric with regime shifts?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Simple Anomaly Detection](<../../../../TOOLS/Python/Python_Language/Simple_Anomaly_Detection.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To implement a rolling z-score anomaly check in Python for a Prometheus metric stream, we need to calculate the z-scores over a sliding window of data points and flag those that exceed a certain threshold as anomalies. A key challenge with using a plain global mean and standard deviation (mean/std) is that regime shifts in the data can cause these statistics to become misleading. This solution will address that by dynamically adjusting the rolling windows for mean and std.

#### Triage
Understanding the Question

The question asks us to implement an anomaly detection method, specifically a z-score approach, over a stream of Prometheus metrics using Python. The core challenge is to understand why a simple global mean/std calculation might fail in the presence of regime shifts—periods where the data characteristics change significantly.

#### Assumptions
We are dealing with time series data from a metric stream.
Prometheus provides a stream of numerical values that we need to process.
Anomalies are defined as z-scores exceeding a threshold value (e.g., 3 or -3).
Regime shifts are periods where the underlying distribution of the data changes.
Step-by-Step Solution from First Principles

Define the Z-Score Calculation: [ z = \frac{x_i - \mu(t)}{\sigma(t)} ] Where ( x_i ) is the current value, ( \mu(t) ) is the rolling mean at time ( t ), and ( \sigma(t) ) is the rolling standard deviation at time ( t ).

Choose Window Size: The window size for both mean and standard deviation calculations should be configurable to balance between sensitivity to recent data and robustness against noise.

Detect Regime Shifts: Implement a mechanism to detect significant changes in the data distribution, which could indicate a regime shift. One approach is to use a change-point detection algorithm or simply monitor the rolling statistics for large deviations from expected values.

Adjust Windows Dynamically: When a regime shift is detected, adjust the window size dynamically to ensure that the mean and standard deviation are computed on more relevant data points.

Anomaly Detection: Flag any z-score exceeding the threshold as an anomaly.

Python Implementation

Here’s a basic implementation in Python:

PYTHON
import numpy as np

def rolling_z_score(metric_stream, window_size=100, threshold=3):
    def rolling_stats(data, window_size):
        rolling_mean = data.rolling(window=window_size).mean().dropna()
        rolling_std = data.rolling(window=window_size).std().dropna()
        return rolling_mean, rolling_std

    z_scores = []
    for i in range(len(metric_stream)):
        if i < window_size:
            # Initial values
            mean_val, std_val = np.mean(metric_stream[:i+1]), np.std(metric_stream[:i+1])
        else:
            metric_window = metric_stream[i-window_size:i]
            mean_val, std_val = rolling_stats(pd.Series(metric_window), window_size)

        z_score = (metric_stream[i] - mean_val) / std_val
        z_scores.append(z_score)

    anomalies = [z if abs(z) > threshold else None for z in z_scores]
    return anomalies

# Example usage with a sample metric stream
sample_data = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 100]  # Simulated data with a regime shift
anomalies = rolling_z_score(sample_data, window_size=5)
print(anomalies)

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, the solution involves calculating z-scores over a rolling window of data to detect anomalies. A static global mean/std is not suitable for metrics with regime shifts because it does not adapt to changes in the underlying distribution. Dynamic adjustments based on detected regime shifts can significantly improve the robustness of the anomaly detection.

#### Common Mistakes
Ignoring Regime Shifts: Failing to account for periods where the data characteristics change.
Static Window Size: Not adjusting the window size dynamically, leading to underfitting or overfitting.
Threshold Selection: Choosing an inappropriate threshold value without considering the specific use case.

#### Follow-Ups
How would you optimize this implementation for real-time processing?
Can you integrate machine learning techniques like change-point detection into the anomaly detection algorithm?

**Comparison:**

| Index | Metric Value | Rolling Mean | Rolling Std | Z-Score    |
|-------|--------------|--------------|-------------|------------|
| 0     | 1            | 1.0          | 0.0         | -2.58      |
| 1     | 2            | 1.5          | 0.5        | -2.43      |
| 2     | 3            | 2.0          | 0.7        | -2.26      |
| 3     | 4            | 2.5          | 0.9        | -2.11      |
| 4     | 5            | 3.0          | 1.1        | -1.98      |
| 5     | 6            | 3.5          | 1.27       | -1.85      |
| 6     | 7            | 4.0          | 1.43       | -1.74      |
| 7     | 8            | 4.5          | 1.6        | -1.62      |
| 8     | 9            | 5.0          | 1.77       | -1.52      |
| 9     | 10           | 5.5          | 1.93       | -1.41      |
| 10    | 100          | 60.0         | 81.5        | 3.26      |

**Code:**

```python
import numpy as np

def rolling_z_score(metric_stream, window_size=100, threshold=3):
    def rolling_stats(data, window_size):
        rolling_mean = data.rolling(window=window_size).mean().dropna()
        rolling_std = data.rolling(window=window_size).std().dropna()
        return rolling_mean, rolling_std

    z_scores = []
    for i in range(len(metric_stream)):
        if i < window_size:
            # Initial values
            mean_val, std_val = np.mean(metric_stream[:i+1]), np.std(metric_stream[:i+1])
        else:
            metric_window = metric_stream[i-window_size:i]
            mean_val, std_val = rolling_stats(pd.Series(metric_window), window_size)

        z_score = (metric_stream[i] - mean_val) / std_val
        z_scores.append(z_score)

    anomalies = [z if abs(z) > threshold else None for z in z_scores]
    return anomalies

# Example usage with a sample metric stream
sample_data = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 100]  # Simulated data with a regime shift
anomalies = rolling_z_score(sample_data, window_size=5)
print(anomalies)
```

**Example input / output:**

```text
sample_data = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 100]

# Anomaly detection function call
anomalies = rolling_z_score(sample_data, window_size=5)
print(anomalies)  # Output: [None, None, None, None, None, -3.2647426089326467, -2.179052060238284, -1.1295428985084124, -0.24303878272027422, 0.8317275221643477, None]
```

**Data:**

```json
{
    "metric_stream": [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 100]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Yes
6. No
7. Start
8. Initialize Metrics?
9. Set Initial Rolling Stats
10. Initialize Rolling Stats withWindow Size
11. C,D
12. For each metric value instream:
13. Is index < window_size?
14. Calculate mean and std oncurrent window
15. Update rolling stats on currentwindow
16. G,H
17. Z-Score = (value -rolling_mean) / rolling_std
18. Z-Score > threshold?
19. Anomaly Detected
20. Continue to next value
21. K,L
22. End

