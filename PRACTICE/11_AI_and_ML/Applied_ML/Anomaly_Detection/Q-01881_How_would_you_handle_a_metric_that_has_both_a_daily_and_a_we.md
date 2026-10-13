---
type: "practice"
difficulty: "Deep"
---

# How would you handle a metric that has both a daily and a weekly seasonal pattern?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Seasonality and Trend Handling](<../../../../11_AI_and_ML/Applied_ML/Anomaly_Detection/Seasonality_and_Trend_Handling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address a time series with both daily and weekly seasonality, we need to design an anomaly detection system that can effectively model these patterns. The key is to use a robust framework that can separate out the seasonal components before detecting anomalies. Here's a step-by-step approach:

Understanding the Question: The interviewer wants you to describe how to handle time series data with multiple seasonal patterns (daily and weekly). They are probing your understanding of advanced anomaly detection techniques, particularly those capable of handling complex seasonality.

Key Assumptions: We assume the metric is continuous and periodic in nature, with daily and weekly trends being the dominant features. We will use a combination of statistical models to handle both patterns.

#### First-Principles Solution

Decompose the Time Series: Start by decomposing the time series into its components: trend, seasonalities (daily and weekly), and residuals.
Use Additive Decomposition Model: Apply an additive decomposition model that can capture both daily and weekly seasonality. This involves subtracting out the effects of these patterns to isolate the underlying trend and noise.
Model Each Seasonality Separately: Use a seasonal ARIMA (SARIMA) or similar model to fit each type of seasonality separately, then combine them.
Detect Anomalies: Once the seasonality is removed, use statistical methods like Z-score or modified Z-score to detect anomalies in the residuals.

Detailed Steps:

Step 1: Data Preprocessing

PYTHON
from pandas import read_csv
from statsmodels.tsa.seasonal import seasonal_decompose

# Load data
df = read_csv('metric_data.csv', parse_dates=['timestamp'], index_col='timestamp')

# Decompose time series
result = seasonal_decompose(df['value'])
trend = result.trend
daily_seasonality = result.seasonal['daily']
weekly_seasonality = result.seasonal['weekly']

df_trend = pd.DataFrame(trend, columns=['trend'])
df_daily = pd.DataFrame(daily_seasonality, columns=['daily_seasonality'])
df_weekly = pd.DataFrame(weekly_seasonality, columns=['weekly_seasonality'])

# Combine back
combined_df = pd.concat([df_trend, df_daily, df_weekly], axis=1)

Step 2: Model Seasonalities

PYTHON
from statsmodels.tsa.statespace.sarimax import SARIMAX

# Define and fit models for daily and weekly seasonality
model_daily = SARIMAX(combined_df['trend'], exog=combined_df[['daily_seasonality']], order=(1, 0, 0))
results_daily = model_daily.fit()

model_weekly = SARIMAX(combined_df['trend'], exog=combined_df[['weekly_seasonality']], order=(1, 0, 0))
results_weekly = model_weekly.fit()

Step 3: Remove Seasonality

PYTHON
# Predict and subtract seasonality from the original data
predicted_daily = results_daily.forecast(steps=len(df), exog=combined_df[['daily_seasonality']])
predicted_weekly = results_weekly.forecast(steps=len(df), exog=combined_df[['weekly_seasonality']])

combined_df['trend_no_seasonalities'] = df['value'].subtract(predicted_daily).subtract(predicted_weekly)

Step 4: Detect Anomalies

PYTHON
# Calculate Z-score for the residuals
from scipy.stats import zscore

combined_df['residuals'] = zscore(combined_df['trend_no_seasonalities'])

# Flag anomalies based on a threshold
combined_df['anomaly'] = (combined_df['residuals'].abs() > 3)

#### Closing Summary
and Common Mistakes/Follow-Ups:

Summary: By decomposing the time series into its seasonal components, we can effectively isolate the anomalies from the regular patterns. This approach ensures that both daily and weekly seasonality are accounted for.

#### Common Mistakes
One common mistake is not adequately separating out the seasonalities before detecting anomalies. Another is using a single model to fit all types of seasonality without considering their interactions.

#### Follow-Ups

What other models could be used instead of SARIMA?
How would you handle missing data in your time series?
Can you discuss the trade-offs between complexity and accuracy in this approach?

**Comparison:**

| timestamp       | value | daily_seasonality | weekly_seasonality | trend_no_seasonalities | residuals | anomaly |
|-----------------|-------|-------------------|--------------------|------------------------|-----------|---------|
| 2023-10-01      | 5.6   | 1.2               | -0.8               | 4.2                    | 0.7       | False   |
| 2023-10-02      | 6.1   | 0.9               | -0.7               | 5.0                    | 0.6       | False   |
| ...             | ...   | ...               | ...                | ...                    | ...       | ...     |

**Code:**

```python
from pandas import read_csv
from statsmodels.tsa.seasonal import seasonal_decompose
from statsmodels.tsa.statespace.sarimax import SARIMAX
from scipy.stats import zscore
```

**Example input / output:**

```text
# Input Data
df = pd.DataFrame({
    'timestamp': pd.date_range(start='2023-10-01', periods=30),
    'value': [5.6, 6.1, 4.8, 7.2, ...],
    'daily_seasonality': [1.2, 0.9, -0.5, ..., -0.3],
    'weekly_seasonality': [-0.8, -0.7, -0.4, ..., -0.2]
})

# Output Data
combined_df = pd.concat([df, df_trend, df_daily, df_weekly], axis=1)
combined_df['trend_no_seasonalities'] = df['value'].subtract(predicted_daily).subtract(predicted_weekly)
combined_df['residuals'] = zscore(combined_df['trend_no_seasonalities'])
combined_df['anomaly'] = (combined_df['residuals'].abs() > 3)

# Output Anomalies
print(combined_df[combined_df['anomaly']])
```

**Data:**

```
CSV
Copy
timestamp,value,daily_seasonality,weekly_seasonality,trend_no_seasonalities,residuals,anomaly
2023-10-01,5.6,1.2,-0.8,4.2,0.7,False
2023-10-02,6.1,0.9,-0.7,5.0,0.6,False
...
```

**Flow:**

1. Yes
2. Yes
3. No
4. Load Data
5. Decompose Time Series
6. Daily Seasonality?
7. Model Daily Seasonality
8. Weekly Seasonality?
9. Model Weekly Seasonality
10. Combine Models
11. Remove Seasonalities
12. Z-Score Anomalies
13. Detect Anomalies

