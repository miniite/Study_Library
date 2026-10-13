---
type: "practice"
difficulty: "Expert"
---

# Your Python anomaly detector's false-positive rate spiked after a legit traffic pattern change (a new feature launch). How would you distinguish a permanent regime change from a transient anomaly and adapt the baseline?

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ Change-point Detection](<../../../../TOOLS/Python/Python_Language/Change-point_Detection.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we need to distinguish between a genuine shift in traffic patterns due to a feature launch and transient anomalies that may be causing false positives in our Python-based anomaly detector. The key challenge is adapting our baseline to account for the regime change while minimizing false positives.

#### Triage

The question is probing how we can dynamically adjust our anomaly detection algorithm to accommodate changes in data characteristics, particularly focusing on distinguishing between a one-time regime shift and ongoing anomalies. We need to consider the nature of the traffic patterns, the frequency of these changes, and potential thresholds for identifying such shifts.

#### Assumptions

The system has historical data from which baselines can be derived.
Transient anomalies are noise that is not part of the underlying pattern.
Permanent regime changes are structural shifts in the data distribution.

#### First Principles Solution
Data Preparation: Ensure we have a robust and complete dataset, including recent data to capture the new feature launch impact.
Baseline Calculation: Use historical data to calculate initial baselines for normal behavior.
Anomaly Detection Algorithm: Implement an algorithm that can flag potential anomalies based on deviations from these baselines.
Change Point Detection: Develop a method to detect when there might be a shift in the underlying data distribution, indicating a permanent regime change.
Adaptive Baseline Adjustment: Update the baselines periodically to account for any detected changes.
Detailed Solution

Data Preparation:

Collect and preprocess historical traffic data covering a sufficient time period before and after the feature launch.
Ensure the dataset includes enough variability to capture both normal behavior and potential anomalies.

Baseline Calculation:

Use statistical methods like moving averages, rolling windows, or exponentially weighted moving averages (EWMA) to establish baselines for typical traffic patterns.
PYTHON
import numpy as np

def calculate_moving_average(data, window=30):
    return np.convolve(data, np.ones(window), 'valid') / window

Anomaly Detection Algorithm:

Define thresholds based on the calculated moving averages to identify deviations that may indicate anomalies.
PYTHON
def detect_anomalies(data, baseline, threshold=2):
    return abs((data - baseline) / baseline) > threshold

Change Point Detection:

Use statistical tests like the Page-Hinkley test or cumulative sum (CUSUM) to detect shifts in the data.
PYTHON
def page_hinkley(data, delta=0.1, lambda_value=0.95):
    z = 0
    for x in data:
        z += x - delta
        if z > lambda_value * z:
            return True
    return False

Adaptive Baseline Adjustment:

Periodically recompute the baseline when a change point is detected.
PYTHON
def update_baseline(baseline, new_data):
    # Use EWMA to update the baseline with new data
    alpha = 0.1
    new_baseline = (1 - alpha) * baseline + alpha * new_data
    return new_baseline

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, distinguishing between permanent regime changes and transient anomalies requires a robust approach that includes dynamic baselines and adaptive algorithms. Common pitfalls include overfitting to noise or failing to adapt quickly enough to significant shifts in the data.

#### Follow-ups
could involve discussing more advanced techniques like Bayesian changepoint detection or incorporating machine learning models for better anomaly detection.

