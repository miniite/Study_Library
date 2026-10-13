---
type: "practice"
difficulty: "Deep"
---

# What would you check first when precision looks great in backtests but the detector spams noise in production?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Adaptive Thresholds](<../../../../11_AI_and_ML/Applied_ML/Anomaly_Detection/Adaptive_Thresholds.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When an anomaly detection system performs well in backtests with high precision but starts spamming noise in production, it indicates a critical issue that needs to be thoroughly investigated. The first step is to understand why the model might behave differently in production compared to backtesting. This discrepancy could stem from several factors including data drift, environmental changes, or inadequate handling of seasonal patterns and trends.

To diagnose this, we need to start by checking a few key areas:

Data Quality and Drift: Verify if there are any changes in the input data that might not have been accounted for during backtesting.
Model Configuration: Ensure that the model parameters remain consistent between testing and production environments.
Temporal Patterns: Examine whether the underlying temporal patterns, such as seasonality or trends, have changed significantly.

Let's break down each of these areas in detail:

1. Data Quality and Drift

First, we should inspect the data quality metrics to ensure that there is no missing data, outliers, or any other anomalies that could affect the model’s performance. Use tools like statistical tests (e.g., Z-score) or visualizations (box plots) to identify these issues.

Inspect Data Quality
Identify Missing Values
Outliers and AnomaliesDetected

Next, we need to check for data drift. This involves comparing the distribution of training data with production data. Tools like MLflow or custom scripts can be used to perform these comparisons.

2. Model Configuration

Ensure that all model parameters are set correctly in both environments. Common issues include differences in hyperparameters, feature engineering steps, and preprocessing pipelines.

Model Parameters
Verify Hyperparameters
Feature Engineering Steps
Preprocessing Pipeline
3. Temporal Patterns

Given that anomaly detection is often time-series oriented, it’s crucial to verify if the temporal patterns (seasonality and trends) have changed significantly.

Temporal Patterns
Seasonality Changed
Trends Altered

To further investigate these temporal changes, you can use techniques like moving averages or decompose time series to isolate seasonality and trend components. Tools like statsmodels in Python can help with this.

#### Closing Summary

By systematically addressing each of these areas—data quality, model configuration, and temporal patterns—we can identify the root cause of the issue. It’s essential to maintain robust monitoring systems that continuously check for data drift and performance degradation to prevent such issues from arising.

#### Common mistakes
often involve overlooking subtle changes in production data or assuming that backtest results directly translate to real-world scenarios without proper validation.

**Comparison:**

```
Parameter	Train	Production
Mean	3	12
Standard Dev.	1.56	4.78
Minimum	1	10
Maximum	5	14
```

**Code:**

```python
# Example code snippet for checking data quality and drift
import pandas as pd
from sklearn.metrics import mean_squared_error

def check_data_quality(df_train, df_production):
    # Check missing values
    print("Missing Values in Train:", df_train.isnull().sum())
    print("Missing Values in Prod:", df_production.isnull().sum())

    # Perform outlier detection
    q1 = df_train.quantile(0.25)
    q3 = df_train.quantile(0.75)
    iqr = q3 - q1
    lower_bound = q1 - 1.5 * iqr
    upper_bound = q3 + 1.5 * iqr

    # Identify outliers in train and production data
    print("Outliers in Train:", (df_train[(df_train < lower_bound) | (df_train > upper_bound)].count()))
    print("Outliers in Prod:", (df_production[(df_production < lower_bound) | (df_production > upper_bound)].count()))

def check_model_parameters(model):
    # Example: Check hyperparameters
    print("Hyperparameters:", model.get_params())

def check_temporal_patterns(df_train, df_production):
    from statsmodels.tsa.seasonal import seasonal_decompose

    # Decompose time series for seasonality and trends
    result_train = seasonal_decompose(df_train['value'], model='additive')
    result_prod = seasonal_decompose(df_production['value'], model='additive')

    print("Seasonal Component Train:", result_train.seasonal)
    print("Trend Component Train:", result_train.trend)
    print("Seasonal Component Prod:", result_prod.seasonal)
    print("Trend Component Prod:", result_prod.trend)

# Sample data
df_train = pd.DataFrame({'value': [1, 2, 3, 4, 5]})
df_production = pd.DataFrame({'value': [10, 11, 12, 13, 14]})

check_data_quality(df_train, df_production)
check_model_parameters(<model>)
check_temporal_patterns(df_train, df_production)
```

**Example input / output:**

| Action | Input Data                 | Output                          |
|--------|----------------------------|--------------------------------|
| Check  | `df_train` & `df_production` | Missing Values, Outliers, Temporal Components |

**Data:**

```json
{
    "df_train": {
        "value": [1, 2, 3, 4, 5]
    },
    "df_production": {
        "value": [10, 11, 12, 13, 14]
    }
}
```

**Flow:**

1. Inspect Data Quality
2. Identify Missing Values
3. Outliers and AnomaliesDetected
4. Verify Model Parameters
5. Review Hyperparameters
6. Examine Feature EngineeringSteps
7. Validate Preprocessing Pipeline
8. Temporal Patterns
9. Seasonality Changed
10. Trends Altered

