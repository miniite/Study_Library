---
type: "practice"
difficulty: "Basic"
---

# What is a baseline in monitoring and how is it used to detect anomalies?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Multivariate Anomaly Correlation](<../../../../11_AI_and_ML/Applied_ML/Anomaly_Detection/Multivariate_Anomaly_Correlation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In AIOps, especially when dealing with time series data, understanding baselines is crucial for anomaly detection. Think of a baseline as a reference point that represents normal behavior in your system over time. This concept helps us distinguish between normal fluctuations and actual anomalies.

#### Triaging the Question

The question is asking you to explain what a baseline is and how it's used in monitoring systems, specifically focusing on anomaly detection for time series data. The interviewer is probing your understanding of fundamental concepts and your ability to articulate technical ideas clearly.

#### First-Principles Solution

A baseline in monitoring contexts represents the expected normal behavior or typical range of values for a particular metric over a period. For time series data, which are sequences of measurements made at regular intervals, this means identifying patterns that have been consistently observed.

Step 1: Define Baseline
Static Baseline: A fixed value or range that does not change over time.
Dynamic Baseline: A moving average or a more sophisticated statistical model that adapts to changes in the data over time.
Step 2: Establishing the Baseline

To establish a baseline, you typically:

Collect historical data for the metric of interest.
Analyze this data using statistical methods (e.g., mean, median, quantiles) or machine learning techniques to derive normal behavior patterns.
Update the baseline periodically as new data becomes available.
Step 3: Detecting Anomalies

Once you have a baseline:

Comparison: Compare current metric values against the established baseline.
Thresholds: Define thresholds for what constitutes an anomaly based on the variability of the data around the baseline.
Statistical Tests: Use statistical methods like Z-scores, P-values, or more advanced models to identify deviations from expected behavior.

#### Closing Summary

In summary, a baseline in time series monitoring represents the normal operating range. It is used as a reference point for detecting anomalies by comparing current values against this established norm. Effective anomaly detection relies on accurately establishing and updating the baseline over time.

#### Common Mistakes/Follow-Ups
Mistake: Using only short-term data to set long-term baselines, leading to poor detection of rare but significant anomalies.

#### Follow-Up
Discussing how different types of baselines (static vs. dynamic) affect anomaly detection in various use cases and the trade-offs involved.

**Comparison:**

```
Time	Value	Moving Average Baseline
1	10	-
2	20	20
3	30	30
4	40	40
5	50	50
6	60	60
7	70	-
8	80	-
9	90	-
```

**Code:**

```python
# Example Python code for a simple moving average baseline
import numpy as np

def calculate_moving_average(data, window=5):
    return np.convolve(data, np.ones(window), 'valid') / window

data = [10, 20, 30, 40, 50, 60, 70, 80, 90]
baseline = calculate_moving_average(data)
print("Moving Average Baseline:", baseline)
```

**Example input / output:**

```text
Input Data: `[10, 20, 30, 40, 50, 60, 70, 80, 90]`

Baseline Calculation (Moving Average with Window=5): `[20, 30, 40, 50, 60]`

### Summary
The provided code calculates a moving average baseline for the given time series data. The resulting baseline helps in comparing new values against this historical norm to detect anomalies.
```

**Data:**

```json
{
  "data": [10, 20, 30, 40, 50, 60, 70, 80, 90],
  "baseline": [20, 30, 40, 50, 60]
}
```

**Flow:**

1. Static
2. Dynamic
3. Yes
4. No
5. Collect Time Series Data
6. Analyze Historical Data
7. Use Static or DynamicBaseline?
8. Establish Fixed Baseline
9. Predictive Model to EstablishBaseline
10. Compare Current ValuesAgainst Baseline
11. Train and Update PredictiveModel Regularly
12. Value Outside Thresholds?
13. Anomaly Detected
14. Normal Operation

