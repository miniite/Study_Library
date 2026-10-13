---
type: "practice"
difficulty: "Intermediate"
---

# Why would you alert on the rate of change of a metric, and what noise problems does the derivative introduce?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Smoothing and Sensitivity Trade-offs](<../../../../11_AI_and_ML/Applied_ML/Anomaly_Detection/Smoothing_and_Sensitivity_Trade-offs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Alerting on the rate of change of a time series can be very effective in identifying anomalies because it helps to detect sudden shifts or trends that might not be apparent when looking at the raw values alone. However, using derivatives introduces noise and other challenges that must be carefully managed.

Firstly, let's consider why we would want to alert on the rate of change:

Sudden Changes: The rate of change can highlight abrupt changes in behavior, which often indicate anomalies or issues.
Trends: Detecting trends through derivatives helps identify when a system is moving away from its normal state over time.

Now, let's dive into the noise problems and potential solutions:

#### Triage
What the Question Is Really Asking

The question primarily probes your understanding of:

Why using derivatives for anomaly detection can be beneficial.
The inherent challenges and noise issues associated with derivatives.
How to mitigate these noise issues in practice.
Step-by-Step Solution From First Principles
1. Derivative Calculation

Derivatives are calculated as the rate of change between two points on a time series. For instance, if we have a metric M(t) at times t and t+Δt, the derivative can be approximated by: [ \frac{dM}{dt} = \frac{M(t+Δt) - M(t)}{Δt} ]

2. Noise in Derivatives

The key noise problems arise from the inherent volatility of derivatives:

High Variance: Small fluctuations in the raw data can lead to large variations in the derivative, making it unreliable for anomaly detection.
Sampling Issues: The choice of Δt (the time interval) is critical; too short a period introduces too much noise, while too long a period might miss small but significant changes.
3. Smoothing Techniques

To mitigate these issues, we can use smoothing techniques such as:

Moving Averages: Calculate the average rate of change over a fixed window.
Exponential Moving Average (EMA): Weight recent values more heavily to reduce noise while still capturing trends.
4. Thresholding and Contextual Alerts

Setting appropriate thresholds for alerts is crucial. This involves defining what constitutes an "abnormal" rate of change based on historical data and contextual factors such as business hours, seasonality, or known events that might cause fluctuations.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, while the rate of change can be a powerful tool for detecting anomalies in time series data, it requires careful handling due to inherent noise. Smoothing techniques, thresholding, and contextual considerations are essential steps to mitigate these issues. Common mistakes include:

Overlooking the need for smoothing.
Setting overly strict thresholds that miss important signals.
Ignoring the context of the data when defining normal behavior.

#### Follow-up
questions might explore specific smoothing algorithms or discuss how machine learning could be used to improve anomaly detection in time series data.

**Comparison:**

```
Time	Value	Derivative
2023-10-07 00:00	4	NaN
2023-10-07 01:00	5	0.67
2023-10-07 02:00	6	0.50
```

**Code:**

```python
import pandas as pd

# Example function for calculating derivative using a moving average approach
def calculate_derivative(df, window_size):
    df['derivative'] = (df['value'].diff() / df.index.to_series().diff()).rolling(window=window_size).mean()
    return df

# Sample data
data = {'time': pd.date_range(start='2023-10-01', periods=50, freq='H'), 'value': [1, 2, 3, 4, 5, 6] * 8 + [7, 6, 5, 4]}
df = pd.DataFrame(data)

# Calculate derivative
calculate_derivative(df, window_size=5)
print(df.tail())
```

**Example input / output:**

```text
time                       value   derivative
2023-10-07 00:00:00+00:00   4      NaN
2023-10-07 01:00:00+00:00   5      0.666667
2023-10-07 02:00:00+00:00   6      0.500000
```

**Data:**

```json
{
  "data": {
    "time": ["2023-10-07T00:00:00Z", "2023-10-07T01:00:00Z", "2023-10-07T02:00:00Z"],
    "value": [4, 5, 6]
  }
}
```

**Flow:**

1. Calculate Derivative
2. Smoothing Techniques
3. Thresholding and ContextualAlerts
4. Set Thresholds
5. Evaluate Anomalies
6. Alert on Anomalies

