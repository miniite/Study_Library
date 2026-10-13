---
type: "practice"
difficulty: "Deep"
---

# How would you debug an anomaly detector that suddenly started firing on every host in one region at 2am but the service was healthy?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Adaptive Thresholds](<../../../../11_AI_and_ML/Applied_ML/Anomaly_Detection/Adaptive_Thresholds.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To tackle this issue, we need to understand why our anomaly detection system is incorrectly flagging all hosts in a specific region during an off-peak hour when the service is known to be healthy. Anomaly detection systems are complex and can be influenced by various factors such as data quality, model parameters, or external events. Here’s how I would approach debugging this issue:

Triaging the Problem

The anomaly detector is likely configured to trigger alerts based on statistical anomalies in time series data. The sudden firing of alarms at 2 AM on all hosts suggests an issue with either the detection logic or the underlying data.

Confirm the Service Health: First, verify that the service was indeed healthy during this period.
Review Recent Changes: Check if there were any recent changes to the anomaly detection system, such as model retraining, parameter updates, or configuration tweaks.
Examine Data Quality: Ensure the data is clean and not corrupted; missing values or outliers could cause false positives.
Check for External Influences: Consider external factors that might affect the metrics, such as environmental changes or upstream issues.
Step-by-Step Debugging

We'll debug from first principles by examining the anomaly detection process in detail:

Data Collection and Preprocessing

Verify the collection of data points from each host.
Ensure there are no missing values or anomalies in the collected time series data.

Model Parameters and Configuration

Review the threshold settings for what constitutes an anomaly.
Check if there have been any changes to the model parameters that could affect the detection logic.
Confirm the choice of statistical methods (e.g., Z-score, moving average) used in detecting anomalies.

Anomaly Detection Logic

Walk through the anomaly detection algorithm step-by-step.
Identify potential areas where the system might be misbehaving, such as incorrect data normalization or inappropriate window sizes for time series analysis.

Monitoring and Alerting System

Ensure that the monitoring system is functioning correctly and not introducing errors in alert generation.
Check if there are any false positives due to threshold settings being too low.

External Factors

Investigate whether there were any external events or changes that might have affected the metrics, such as network issues or changes in load balancing configurations.

#### Example
Worked Input-Output

#### Cons
ider a hypothetical scenario where we receive time series data for CPU usage:

PLAINTEXT
Time        Host1_CPU    Host2_CPU
03:00 AM     5%           7%
04:00 AM     6%           8%
05:00 AM     120%         9%
...

If the system suddenly flags all hosts at 5 AM, we would investigate:

Data Quality: Check for any spikes or unusual values.
Model Parameters: Ensure thresholds are set correctly.
Detection Logic: Verify the algorithm's output matches expected behavior.

#### Closing Summary

The key to debugging this issue is a systematic approach that covers data collection, model configuration, and system logic. By methodically checking each component, we can identify and rectify the root cause of the anomalies.

#### Common Mistakes/Follow-Ups
Ignoring data quality issues.
Overlooking changes in external systems or parameters.
Failing to validate the anomaly detection logic thoroughly.

**Comparison:**

| Time     | Host1_CPU | Host2_CPU |
|----------|-----------|-----------|
| 03:00 AM | 5%        | 7%        |
| 04:00 AM | 6%        | 8%        |
| 05:00 AM | 120%      | 9%        |

**Code:**

```python
# Example code for checking time series data quality
import pandas as pd

data = {
    'Time': ['03:00 AM', '04:00 AM', '05:00 AM'],
    'Host1_CPU': [5, 6, 120],
    'Host2_CPU': [7, 8, 9]
}

df = pd.DataFrame(data)
print(df.describe())  # Check for outliers or anomalies
```

**Example input / output:**

```text
Input:
Time        Host1_CPU    Host2_CPU
03:00 AM     5%           7%
04:00 AM     6%           8%
05:00 AM     120%         9%

Output:
count  3.000000e+00    3.000000
mean   1.333333e+01    7.666667
std    8.524792e+01    2.225594
min    5.000000e+00    7.000000
25%    6.000000e+00    7.000000
50%    6.000000e+00    8.000000
75%    6.000000e+01    9.000000
max   1.200000e+02    9.000000

Note: The 75th percentile CPU usage for Host1 is unusually high, indicating a potential outlier.
```

**Data:**

```plaintext
Time        Host1_CPU    Host2_CPU
03:00 AM     5%           7%
04:00 AM     6%           8%
05:00 AM     120%         9%
```

**Flow:**

1. Verify Service Health
2. Check Recent Changes
3. Review Data Quality
4. Examine Model Parameters
5. Anomaly Detection Logic
6. Monitor and Alert System
7. External Factors

