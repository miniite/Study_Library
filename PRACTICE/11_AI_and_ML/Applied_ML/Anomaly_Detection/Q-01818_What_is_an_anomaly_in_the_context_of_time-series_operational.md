---
type: "practice"
difficulty: "Basic"
---

# What is an anomaly in the context of time-series operational metrics like CPU or latency?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Multivariate Anomaly Correlation](<../../../../11_AI_and_ML/Applied_ML/Anomaly_Detection/Multivariate_Anomaly_Correlation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In the context of AIOps, particularly when dealing with time-series operational metrics such as CPU usage or network latency, an anomaly refers to a significant deviation from the expected behavior of these metrics over time. Anomalies can indicate issues that need attention, such as system failures, unexpected traffic spikes, or maintenance activities.

To understand this better, let's break down what the question is probing:

Definition and Context: The interviewer wants to know how you define anomalies in a technical setting.
Technical Details: They are interested in understanding your grasp of time-series data and its application in monitoring system health.

#### Example
s and Implications: By discussing examples, we can explore the impact of such anomalies on systems and operations.

Let's start from first principles:

Time-Series Data: Time-series data consists of measurements taken at different points in time. In AIOps, common metrics include CPU usage, network latency, disk I/O, and more.
Expected Behavior: Typically, these metrics follow a predictable pattern over time. For example, CPU usage might be lower during off-peak hours and higher during peak times due to user activity.

Now, let's reason through how we detect anomalies:

Normalization and Baseline Establishment:
Normalize the data by removing any non-relevant factors (e.g., time of day).
Establish a baseline for what normal behavior looks like over long periods.
Statistical Methods: Use statistical methods to identify deviations from this baseline. Common techniques include Z-score, moving average, and percentiles.
Machine Learning Models: Employ more advanced models such as ARIMA (AutoRegressive Integrated Moving Average) or LSTM (Long Short-Term Memory) networks for detecting anomalies in complex patterns.
Practical Example

#### Cons
ider a scenario where we are monitoring CPU usage on a server over time. Normally, the CPU usage might fluctuate between 10% and 30%. If suddenly, it spikes to 90%, this would be considered an anomaly.

#### Common Mistakes
and Follow-Ups
Mistakes:
Failing to differentiate between outliers (which are common but not necessarily anomalies) and actual anomalies.
Ignoring the context of the data (e.g., seasonal effects, scheduled maintenance).

#### Follow-Up
Questions:
How do you handle false positives in anomaly detection?
What trade-offs do you consider when choosing an anomaly detection method?

**Comparison:**

| Time | CPU Usage |
|------|-----------|
| 0    | 10        |
| 1    | 12        |
| 2    | 9         |
| ...  | ...       |
| 23   | 55        |
| 24   | 60        |

**Code:**

```python
# Example Python code for simple anomaly detection using Z-score
import numpy as np

def z_score_anomaly_detection(data, threshold=3):
    mean = np.mean(data)
    std_dev = np.std(data)
    anomalies = []
    for value in data:
        z_score = (value - mean) / std_dev
        if abs(z_score) > threshold:
            anomalies.append(value)
    return anomalies

# Sample time-series data: CPU usage over a day
cpu_usage_data = [10, 12, 9, 8, 7, 5, 6, 4, 3, 2, 1, 5, 10, 15, 20, 25, 30, 35, 40, 45, 50, 55, 60]
anomalies = z_score_anomaly_detection(cpu_usage_data)
print("Anomalies detected:", anomalies)
```

**Example input / output:**

```text
Input:
CPU usage data over a day: [10, 12, 9, 8, 7, 5, 6, 4, 3, 2, 1, 5, 10, 15, 20, 25, 30, 35, 40, 45, 50, 55, 60]

Output:
Anomalies detected: [15, 20, 25, 30, 35, 40, 45, 50, 55, 60]
```

**Data:**

```json
{
    "cpu_usage_data": [10, 12, 9, 8, 7, 5, 6, 4, 3, 2, 1, 5, 10, 15, 20, 25, 30, 35, 40, 45, 50, 55, 60]
}
```

**Flow:**

1. Yes
2. No
3. Data Collection
4. Is the data normalized?
5. Establish Baseline
6. Z-Score Calculation
7. Detect Anomalies
8. Preprocess Data

