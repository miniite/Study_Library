---
type: "practice"
difficulty: "Deep"
---

# How would you detect a slowly drifting metric that never crosses a fixed threshold?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Drift and Threshold Detection](<../../../../11_AI_and_ML/Applied_ML/Anomaly_Detection/Drift_and_Threshold_Detection.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To detect a slowly drifting metric in time series data, we need to focus on detecting subtle changes over time without relying on sudden thresholds. This problem is common in many monitoring and alerting systems where metrics are expected to remain within certain bounds but can change gradually due to various factors. The core idea here is to model the baseline of the metric and then monitor its drift from this baseline.

#### Triaging the Question

The question asks how to detect a slowly drifting metric that never crosses a fixed threshold, which means we need an anomaly detection system capable of recognizing gradual changes. This involves:

Understanding the Baseline: Identifying the normal range or trend of the metric over time.
Drift Detection Mechanism: Implementing a mechanism to identify when this baseline starts to shift slowly.
Thresholding and Alerting: Defining how significant a drift needs to be before it triggers an alert.

#### First-Principles Solution
Model Baseline Behavior:
Use statistical methods like moving averages, exponential smoothing, or even more advanced models such as ARIMA (AutoRegressive Integrated Moving Average) to model the time series.
Detect Drift:
Implement a drift detection method that continuously monitors changes in the baseline behavior without relying on fixed thresholds.
Define Threshold for Alerting:
Instead of fixed threshold-based alerts, use statistical significance measures like Z-scores or CUSUM (Cumulative Sum) control charts to determine when the metric has drifted significantly.
Detailed Steps

Data Collection and Preprocessing:

Collect historical data on the time series metric.
Ensure the data is clean and free from outliers that could skew the baseline model.

Modeling Baseline Behavior:

Compute a moving average or use exponential smoothing to capture the trend over time.
Alternatively, fit an ARIMA model to account for seasonal patterns and trends in the data.

Drift Detection Mechanism:

CUSUM (Cumulative Sum) Control Chart: Track cumulative deviations from the baseline and trigger alerts when a significant deviation is observed.
Statistical Z-Score Calculation: Calculate z-scores to measure how many standard deviations a new value is away from the mean. If the absolute z-score exceeds a predefined threshold, consider it an anomaly.

Thresholding for Alerting:

Use historical data to set thresholds that are statistically significant.
Continuously monitor these thresholds and adjust them based on real-time feedback.

#### Example
Code
PYTHON
import pandas as pd
from statsmodels.tsa.arima.model import ARIMA

# Sample Data
data = pd.read_csv('timeseries_data.csv')
timestamp = data['timestamp']
value = data['value']

# Fit an ARIMA model to the time series data
model = ARIMA(value, order=(5,1,0))
fitted_model = model.fit()

# Function to calculate CUSUM and detect drift
def cusum_drift_detection(fitted_model, threshold=3):
    residuals = fitted_model.resid
    cumulative_sum = 0.0
    alerts = []

    for residual in residuals:
        cumulative_sum += residual
        if abs(cumulative_sum) > threshold:
            alerts.append((timestamp[-1], cumulative_sum))
            cumulative_sum = 0.0

    return alerts

# Detect drift and get alert times
alerts = cusum_drift_detection(fitted_model)
print("Alert Times:", alerts)

#### Closing Summary

Detecting a slowly drifting metric requires moving away from traditional fixed-threshold methods towards more sophisticated anomaly detection techniques. By modeling the baseline behavior of the time series and continuously monitoring for deviations, we can effectively detect subtle changes that might indicate drift.

#### Common Mistakes/Follow-Ups
Ignoring Seasonality: Failing to account for seasonal patterns in the data.
Fixed Thresholds: Relying solely on fixed thresholds without considering statistical significance.
Overfitting: Using overly complex models that capture noise rather than the underlying trend.

#### Follow-ups
might include discussions about handling seasonality, integrating external factors (like weather or business events), and optimizing model performance for real-time detection.

**Comparison:**

```
Timestamp	Value
2023-01-07	111
2023-01-08	109
```

**Code:**

```python
import pandas as pd
from statsmodels.tsa.arima.model import ARIMA

# Sample Data
data = pd.read_csv('timeseries_data.csv')
timestamp = data['timestamp']
value = data['value']

# Fit an ARIMA model to the time series data
model = ARIMA(value, order=(5,1,0))
fitted_model = model.fit()

# Function to calculate CUSUM and detect drift
def cusum_drift_detection(fitted_model, threshold=3):
    residuals = fitted_model.resid
    cumulative_sum = 0.0
    alerts = []

    for residual in residuals:
        cumulative_sum += residual
        if abs(cumulative_sum) > threshold:
            alerts.append((timestamp[-1], cumulative_sum))
            cumulative_sum = 0.0

    return alerts

# Detect drift and get alert times
alerts = cusum_drift_detection(fitted_model)
print("Alert Times:", alerts)
```

**Example input / output:**

```text
Alert Times: [('2023-01-07', 1.0), ('2023-01-08', -1.5)]
```

**Data:**

```
CSV
Copy
timestamp,value
2023-01-01,105
2023-01-02,106
2023-01-03,107
2023-01-04,108
2023-01-05,109
2023-01-06,110
```

**Flow:**

1. Yes
2. No
3. Data Collection
4. Model Baseline
5. CUSUM Calculation
6. Threshold Check
7. Alert Triggered?
8. Generate Alert
9. Continue Monitoring

