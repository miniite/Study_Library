---
type: "practice"
difficulty: "Deep"
---

# Explain how you would handle a metric with both daily and weekly seasonality plus an upward growth trend in one detector.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Seasonality and Trend Handling](<../../../../11_AI_and_ML/Applied_ML/Anomaly_Detection/Seasonality_and_Trend_Handling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address a metric that exhibits both daily and weekly seasonality along with an upward growth trend, we need to develop a robust anomaly detection system capable of distinguishing between normal variations and true anomalies. The key is to decompose the time series into its components (trend, seasonalities, and residuals), and then define our anomaly detection logic based on these components.

Triaging the Problem

The primary goal here is to design an anomaly detector that can handle multiple types of patterns in a single metric. Specifically, we need to:

Identify the trend: Understand how the metric has been growing over time.
Detect weekly seasonality: Account for regular fluctuations that repeat every week.
Capture daily seasonality: Identify patterns that occur repeatedly on specific days within a week.

The assumptions are as follows:

The data is in time series format with timestamps.
We have historical data available to train our model.
The growth trend and seasonalities might vary over different periods, so we need a flexible approach.

#### First Principles Solution

Decomposition: Use techniques like Seasonal-Trend decomposition using Loess (STL) or Moving Average Smoothing to separate the time series into its components. This will help us isolate each pattern.

Modeling Trends and Seasonality:

For the trend component, a linear regression model can be used if the growth is steady.
For weekly seasonality, we can use Fourier terms or seasonal ARIMA (SARIMA) models.
Daily seasonality might require additional Fourier terms for each day of the week.

Residuals: The residuals after removing these components should represent random noise and potential anomalies.

Anomaly Detection:

Use statistical methods such as Z-score or quantile-based thresholds on the residuals to identify outliers.

#### Cons
ider using machine learning models like Isolation Forests or One-Class SVM for more robust anomaly detection, especially when dealing with complex patterns.

#### Implementation
Steps
Data Preparation: Collect and preprocess time series data, ensuring it is clean and consistent.
Trend Modeling:
PYTHON
import pandas as pd
from statsmodels.tsa.seasonal import seasonal_decompose

# Load dataset
df = pd.read_csv('time_series_data.csv')
ts = df['value']

# Decompose the time series
result = seasonal_decompose(ts, model='additive', period=7)  # Assuming weekly seasonality with a period of 7 days

# Plot decomposed components
result.plot()
Seasonal Modeling:
Use Fourier terms for daily and weekly seasonality.
PYTHON
from statsmodels.tsa.statespace.sarimax import SARIMAX

model = SARIMAX(ts, order=(1, 0, 1), seasonal_order=(1, 1, 1, 7))
results = model.fit()
forecasts = results.get_forecast(steps=24)
forecasted_values = forecasts.predicted_mean
Residuals and Anomaly Detection:
PYTHON
residuals = ts - forecasted_values

# Use Z-score for anomaly detection
z_scores = (residuals - residuals.mean()) / residuals.std()
anomalies = (abs(z_scores) > 3).sum()  # Threshold based on empirical rule

#### Closing Summary
and Common Mistakes/Follow-Ups
Key Takeaways: Decomposition, modeling each component separately, and using statistical methods for anomaly detection.
Common Pitfalls:
Overfitting on the historical data can lead to poor performance when dealing with new data.
Ignoring the trend might result in missing significant anomalies that are influenced by growth.
Not accounting for real-time updates can affect accuracy over time.

#### Follow-Ups
Handling Non-Stationarity: Discuss how to handle non-stationary time series and ensure the model adapts to changing trends.
Real-Time Processing: Explore how the system can be designed for real-time anomaly detection, including streaming data processing frameworks like Apache Flink or Spark Streaming.

**Comparison:**

| Time | Value |
|------|-------|
| 2023-01-01 | 5.2   |
| 2023-01-02 | 5.4   |
| ...  | ...   |
| 2023-12-31 | 7.8   |
| Z-scores |
|----------|
| -1.2     |
| 2.6      |
| ...      |
| 4.1      |

**Code:**

```python
import pandas as pd
from statsmodels.tsa.seasonal import seasonal_decompose
from statsmodels.tsa.statespace.sarimax import SARIMAX

# Load dataset
df = pd.read_csv('time_series_data.csv')
ts = df['value']

# Decompose the time series
result = seasonal_decompose(ts, model='additive', period=7)

# Plot decomposed components
result.plot()

# Model trend and seasonality
model = SARIMAX(ts, order=(1, 0, 1), seasonal_order=(1, 1, 1, 7))
results = model.fit()
forecasts = results.get_forecast(steps=24)
forecasted_values = forecasts.predicted_mean

# Calculate residuals and Z-scores
residuals = ts - forecasted_values
z_scores = (residuals - residuals.mean()) / residuals.std()

# Detect anomalies based on Z-score threshold
anomalies = (abs(z_scores) > 3).sum()
```

**Example input / output:**

```text
Input:
time_series_data.csv

Output:
Decomposed components plotted.
Fitted SARIMAX model with forecasted values.
Anomalies detected based on Z-score threshold of 3.
Number of anomalies found: [number]
```

**Data:**

```plaintext
time_series_data.csv
value
2023-01-01, 5.2
2023-01-02, 5.4
...
2023-12-31, 7.8
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Load Data
6. Decompose Time Series
7. Is Trend Non-Stationary?
8. Trend Modeling with SARIMA
9. Directly Model Seasonality
10. Fitted Model Forecasting
11. Fitted Model Forecasting
12. F,G
13. Calculate Residuals andZ-scores
14. Detect Anomalies?
15. Detect Anomalies
16. No Anomalies Found

