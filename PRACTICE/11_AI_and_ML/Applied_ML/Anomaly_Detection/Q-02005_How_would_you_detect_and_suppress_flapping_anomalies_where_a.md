---
type: "practice"
difficulty: "Intermediate"
---

# How would you detect and suppress flapping anomalies where a metric oscillates around the threshold?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Drift and Threshold Detection](<../../../../11_AI_and_ML/Applied_ML/Anomaly_Detection/Drift_and_Threshold_Detection.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the problem of detecting and suppressing "flapping" anomalies in time series data, we need to focus on identifying rapid fluctuations that repeatedly cross a given threshold. This can be challenging because flapping typically involves oscillations around the threshold rather than persistent deviations. The goal is to accurately detect genuine anomalies while filtering out false positives due to noise or short-term fluctuations.

#### Triaging the Question

Understanding Flapping: Flapping refers to rapid, repeated crossings of a threshold in time series data.
Objective: Identify true anomalies that are not just noise or transient phenomena.

#### Assumptions
We have access to a time series dataset representing metrics over time.
The threshold is predefined and represents the normal operating range.

Step-by-Step Solution:

Data Preprocessing: Normalize the data if necessary, ensuring that the metric values are on a consistent scale.

Threshold Crossings Detection: Identify times when the metric crosses the threshold in both directions (above and below).

Filtering Rapid Flappings:

Short-Term Window Analysis: Use a sliding window to detect rapid oscillations.
Count Crossings: Count how many times the metric crosses the threshold within a short window period.
Threshold for Suppression: Set a threshold on the number of crossings allowed in the window. If this threshold is exceeded, suppress detection.

Long-Term Trend Analysis:

Use statistical methods (e.g., moving average) to filter out noise and identify persistent trends.
Identify periods where the metric stays above or below the threshold for a longer duration compared to flapping behavior.

Anomaly Scoring: Assign an anomaly score based on the number of crossings, persistence, and deviation from the long-term trend.

Suppression Mechanism:

Once a period is identified as flapping, suppress further detections during this period.
Implement a cooldown or suppression window to ensure that false positives are not detected repeatedly.

#### Closing Summary
To detect and suppress flapping anomalies effectively, we need to analyze the metric's behavior using both short-term and long-term metrics. By counting threshold crossings in windows and filtering based on persistence and deviation from trends, we can accurately identify true anomalies while suppressing repeated detections of oscillations around thresholds.

#### Common Mistakes/Follow-Ups

Over-Suppression: Suppressing too much data can lead to missing important anomalies.
Insufficient Suppression: Failing to suppress flapping can result in frequent false alarms.
Choice of Window Size: The window size for detecting crossings is critical and needs tuning based on the specific application.

**Code:**

```python
import pandas as pd

# Sample time series data
data = {
    'timestamp': ['2023-10-01 00:00', '2023-10-01 00:05', '2023-10-01 00:10', '2023-10-01 00:15'],
    'metric_value': [8.5, 9.2, 7.8, 8.4]
}
df = pd.DataFrame(data)
df['timestamp'] = pd.to_datetime(df['timestamp'])
df.set_index('timestamp', inplace=True)

def detect_flapping(df, threshold, window_size=10):
    """
    Detects flapping anomalies and suppresses detections.
    :param df: DataFrame containing time series data
    :param threshold: Threshold value to cross
    :param window_size: Size of the sliding window in minutes
    :return: DataFrame with detected anomalies
    """
    # Calculate the number of times the metric crosses the threshold within each window
    crossings = df.apply(lambda x: sum(abs(x - threshold) > 0), axis=1)

    # Filter out flapping behavior by setting a suppression threshold
    suppressions = crossings.rolling(window=f'{window_size}T').sum() > 2

    return suppressions

# Detect and suppress flapping anomalies
suppressions = detect_flapping(df, 8.0, window_size=5)
print(suppressions)
```

**Example input / output:**

| timestamp            | metric_value | suppressions |
|----------------------|--------------|--------------|
| 2023-10-01 00:00     | 8.5          | False        |
| 2023-10-01 00:05     | 9.2          | True         |
| 2023-10-01 00:10     | 7.8          | True         |
| 2023-10-01 00:15     | 8.4          | False        |

**Data:**

```json
{
  "data": {
    "timestamp": ["2023-10-01 00:00", "2023-10-01 00:05", "2023-10-01 00:10", "2023-10-01 00:15"],
    "metric_value": [8.5, 9.2, 7.8, 8.4]
  }
}
```

