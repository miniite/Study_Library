---
type: "practice"
difficulty: "Expert"
---

# Your team runs an ML model for cloud-cost anomaly detection. Walk me through why a generic z-score or Prophet baseline fails on real billing data (billing-lag, sparse SKUs, step-function commitment amortization) and how you'd architect a model that understands the peculiar shape of cost time-series.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Cost Anomaly Detection](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Cost_Anomaly_Detection.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we need to design an anomaly detection system for cloud costs using historical billing data. A z-score or Prophet baseline fails because these models don't account for critical aspects like billing lag, sparse SKUs, and step-function commitment amortization. Instead, we'll construct a custom model that leverages domain-specific knowledge to better understand the peculiarities of cost time-series.

First, let's break down why traditional approaches fail:

Billing Lag: Cloud providers often bill charges based on events from one month while recording them in another. This can create irregular patterns and outliers.
Sparse SKUs: The number of distinct services (SKUs) is often limited but highly varied, making it difficult for a generic model to learn meaningful patterns.
Step-Function Commitment Amortization: Many cloud resources have fixed monthly costs even when not in use, leading to sudden jumps or flat lines in the cost data.

To address these issues, we need a more nuanced approach that incorporates domain knowledge about how cloud costs are structured. Here’s how we can architect such a model:

#### Triaging the Question

The question is probing for an understanding of why traditional statistical and time-series models like z-score and Prophet fail to capture the complexities of real-world cloud cost data. We need to consider both the technical limitations of these models and the specific characteristics of the data.

Key Assumptions
Data Characteristics: Historical billing data with irregularities due to billing lag, sparse SKUs, and step-function commitment amortization.
Objective: Detect anomalies in cloud costs accurately without false positives or negatives.
Step-by-Step Solution
1. Data Preprocessing

We start by preprocessing the raw billing data:

PYTHON
import pandas as pd

# Example of loading and cleaning billing data
data = pd.read_csv('cloud_billing_data.csv')
data['timestamp'] = pd.to_datetime(data['timestamp'])
data.set_index('timestamp', inplace=True)
2. Feature Engineering

We engineer features that capture the peculiarities of cloud cost data:

PYTHON
def add_features(df):
    df['day_of_week'] = df.index.dayofweek
    df['month'] = df.index.month
    df['hour_of_day'] = df.index.hour
    df['total_commitment'] = df.groupby('sku')['cost'].transform('sum')
    return df

data = add_features(data)
3. Custom Anomaly Detection Model

We design a custom model that accounts for billing lag, sparse SKUs, and step-function commitment amortization:

PYTHON
from sklearn.ensemble import IsolationForest

def train_model(df):
    # Train an Isolation Forest on the preprocessed data
    X = df.drop('cost', axis=1)
    y = df['cost']
    model = IsolationForest(contamination='auto')
    model.fit(X, y)
    return model

model = train_model(data)
4. Model Evaluation and Tuning

We evaluate the model using cross-validation to ensure it generalizes well:

PYTHON
from sklearn.model_selection import TimeSeriesSplit

def evaluate_model(model, df):
    tscv = TimeSeriesSplit(n_splits=5)
    for train_index, test_index in tscv.split(df):
        X_train, y_train = df.iloc[train_index].drop('cost', axis=1), df.iloc[train_index]['cost']
        X_test, y_test = df.iloc[test_index].drop('cost', axis=1), df.iloc[test_index]['cost']
        model.fit(X_train)
        scores = model.decision_function(X_test)
        print(scores)

evaluate_model(model, data)

#### Closing Summary

By carefully preprocessing the data and engineering features that capture the unique characteristics of cloud cost time-series, we can build a robust anomaly detection system. Using machine learning models like Isolation Forests with appropriate feature sets ensures accurate detection even in the presence of billing lags, sparse SKUs, and step-function commitment amortization.

#### Common Mistakes/Follow-Ups
Ignoring Billing Lag: Failing to account for the delay between when a charge occurs and when it is billed can lead to false positives.
Overfitting on Sparse SKUs: Models might struggle with sparse data due to insufficient samples, leading to poor generalization.
Not Handling Step-Function Commitments Properly: Ignoring fixed monthly costs can result in missing anomalies related to resource utilization.

#### Follow-ups
could include asking about handling seasonality, integrating external factors (like AWS service changes), and optimizing the model for real-time performance.

**Comparison:**

| Timestamp    | Day of Week | Month | Hour of Day | Total Commitment |
|--------------|-------------|-------|-------------|------------------|
| 2023-01-05   | 4           | 1     | 8           | 200              |
| 2023-01-06   | 5           | 1     | 9           | 250              |

**Code:**

```python
import pandas as pd

# Example of loading and cleaning billing data
data = pd.read_csv('cloud_billing_data.csv')
data['timestamp'] = pd.to_datetime(data['timestamp'])
data.set_index('timestamp', inplace=True)

def add_features(df):
    df['day_of_week'] = df.index.dayofweek
    df['month'] = df.index.month
    df['hour_of_day'] = df.index.hour
    df['total_commitment'] = df.groupby('sku')['cost'].transform('sum')
    return df

data = add_features(data)

from sklearn.ensemble import IsolationForest

def train_model(df):
    X = df.drop('cost', axis=1)
    y = df['cost']
    model = IsolationForest(contamination='auto')
    model.fit(X, y)
    return model

model = train_model(data)

from sklearn.model_selection import TimeSeriesSplit

def evaluate_model(model, df):
    tscv = TimeSeriesSplit(n_splits=5)
    for train_index, test_index in tscv.split(df):
        X_train, y_train = df.iloc[train_index].drop('cost', axis=1), df.iloc[train_index]['cost']
        X_test, y_test = df.iloc[test_index].drop('cost', axis=1), df.iloc[test_index]['cost']
        model.fit(X_train)
        scores = model.decision_function(X_test)
        print(scores)

evaluate_model(model, data)
```

**Example input / output:**

```text
# Example input data
input_data = pd.DataFrame({
    'timestamp': ['2023-01-05', '2023-01-06'],
    'day_of_week': [4, 5],
    'month': [1, 1],
    'hour_of_day': [8, 9],
    'total_commitment': [200, 250]
})

# Predict using the trained model
input_data.set_index('timestamp', inplace=True)
scores = model.decision_function(input_data)

print(scores) # Output: [-3.14  -4.56] (example scores)
```

**Data:**

```
CSV
Copy
timestamp,cost,sku
2023-01-01 00:00:00,50,AWS_ECS
2023-01-01 06:00:00,40,AWS_S3
2023-01-02 08:00:00,30,AWS_RDS
...
```

**Flow:**

1. Load and Clean Data
2. Add Features
3. Train Isolation Forest Model
4. Evaluate on Cross-Validation
5. Deploy and Monitor Model
6. Maintain and Tune Model

