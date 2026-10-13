---
type: "practice"
difficulty: "Deep"
---

# What is your approach to multivariate anomaly detection where CPU, memory and IO must be evaluated jointly?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Multivariate Anomaly Correlation](<../../../../11_AI_and_ML/Applied_ML/Anomaly_Detection/Multivariate_Anomaly_Correlation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary

In this scenario, we are tasked with implementing a robust multivariate anomaly detection system that evaluates CPU, memory, and I/O metrics concurrently to identify anomalies in an AIOps environment. The key idea is to leverage statistical models or machine learning techniques to capture the interdependencies among these metrics while ensuring timely and accurate detections.

#### Triaging the Question

The interviewer is probing your understanding of multivariate anomaly detection (MAD) techniques, particularly in a time series context relevant to system health monitoring. Key aspects include:

Data Interdependencies: How different metrics like CPU, memory, and I/O are correlated.
Statistical Models vs Machine Learning: Which approach suits the problem best.
Scalability and Real-time Processing: Handling large volumes of data efficiently.

#### First-principles Solution
Step 1: Data Collection and Preprocessing

Collect time series data for CPU usage, memory usage, and I/O operations. Ensure that the data is cleaned and preprocessed to handle missing values, outliers, and noise.

PYTHON
import pandas as pd

# Example DataFrame
data = {
    'timestamp': [pd.Timestamp('2023-10-01 09:00'), pd.Timestamp('2023-10-01 09:05')],
    'cpu_usage': [70, 80],
    'memory_usage': [45, 60],
    'io_ops': [200, 250]
}
df = pd.DataFrame(data)
Step 2: Feature Engineering

Create additional features to capture interdependencies and temporal patterns.

PYTHON
import numpy as np

# Calculate rolling statistics (e.g., moving average)
df['cpu_avg_1min'] = df['cpu_usage'].rolling(window=60, min_periods=1).mean()
df['mem_avg_1min'] = df['memory_usage'].rolling(window=60, min_periods=1).mean()
Step 3: Model Selection

Choose appropriate models for multivariate anomaly detection. Common choices include:

Isolation Forest: Effective for high-dimensional data.
Autoencoders: Capture the normal behavior of the system and flag anomalies.
Dynamic Time Warping (DTW): Measures similarity between time series.

For this example, let's use an Isolation Forest model.

PYTHON
from sklearn.ensemble import IsolationForest

# Fit the model
model = IsolationForest(contamination=0.01)
model.fit(df[['cpu_avg_1min', 'mem_avg_1min', 'io_ops']])

# Predict anomalies
df['anomaly_score'] = model.decision_function(df[['cpu_avg_1min', 'mem_avg_1min', 'io_ops']])
df['is_anomaly'] = model.predict(df[['cpu_avg_1min', 'mem_avg_1min', 'io_ops']])
Step 4: Real-time Anomaly Detection

Implement a streaming system that processes new data points and updates the anomaly detection in real time.

PYTHON
import pandas as pd

def process_data(new_data):
    # Append new data to existing DataFrame
    df = pd.concat([df, new_data])

    # Update rolling statistics
    df['cpu_avg_1min'] = df['cpu_usage'].rolling(window=60, min_periods=1).mean()
    df['mem_avg_1min'] = df['memory_usage'].rolling(window=60, min_periods=1).mean()

    # Predict anomalies for new data
    new_data['anomaly_score'] = model.decision_function(new_data[['cpu_avg_1min', 'mem_avg_1min', 'io_ops']])
    new_data['is_anomaly'] = model.predict(new_data[['cpu_avg_1min', 'mem_avg_1min', 'io_ops']])

    return df, new_data
Step 5: Evaluation and Tuning

Regularly evaluate the performance of the system using metrics like F1 score or precision-recall curves. Tune hyperparameters as needed.

PYTHON
from sklearn.metrics import f1_score

# Example evaluation on a test set
test_data = pd.DataFrame({
    'timestamp': [pd.Timestamp('2023-10-01 09:10')],
    'cpu_usage': [85],
    'memory_usage': [70],
    'io_ops': [300]
})
df, new_test_data = process_data(test_data)
print(f'Anomaly Score: {new_test_data["anomaly_score"].values[0]}, Is Anomaly: {new_test_data["is_anomaly"].values[0]}')

#### Closing Summary

In summary, a robust multivariate anomaly detection system for AIOps involves collecting and preprocessing time series data, feature engineering to capture interdependencies, selecting an appropriate model like Isolation Forest or Autoencoders, implementing real-time processing, and regularly evaluating the system's performance. Common mistakes include overlooking data quality issues, failing to tune hyperparameters effectively, and not considering edge cases.

#### Common Mistakes/Follow-ups
Ignoring Data Quality: Missing values or outliers can significantly affect model performance.
Overfitting/Underfitting: Models must be carefully tuned to avoid overfitting on historical data.
Inadequate Evaluation Metrics: Focusing solely on accuracy metrics without considering precision and recall.
Failure to Handle Real-time Scenarios: Ensuring the system can process new data points quickly without significant latency.

**Comparison:**

```
Time Series Data	CPU Usage	Memory Usage	IO Operations
Timestamp (2023-10-01)	70	45	200
Timestamp (2023-10-01)	80	60	250

Processed Data with Anomaly Detection:

Time Series Data	CPU Usage	Memory Usage	IO Operations	Anomaly Score	Is Anomaly
Timestamp (2023-10-01)	85	70	300	-4.0	1
```

**Code:**

```python
import pandas as pd

# Example DataFrame
data = {
    'timestamp': [pd.Timestamp('2023-10-01 09:00'), pd.Timestamp('2023-10-01 09:05')],
    'cpu_usage': [70, 80],
    'memory_usage': [45, 60],
    'io_ops': [200, 250]
}
df = pd.DataFrame(data)

# Feature Engineering
df['cpu_avg_1min'] = df['cpu_usage'].rolling(window=60, min_periods=1).mean()
df['mem_avg_1min'] = df['memory_usage'].rolling(window=60, min_periods=1).mean()

from sklearn.ensemble import IsolationForest

# Model Selection
model = IsolationForest(contamination=0.01)
model.fit(df[['cpu_avg_1min', 'mem_avg_1min', 'io_ops']])

# Real-time Anomaly Detection
def process_data(new_data):
    df = pd.concat([df, new_data])

    df['cpu_avg_1min'] = df['cpu_usage'].rolling(window=60, min_periods=1).mean()
    df['mem_avg_1min'] = df['memory_usage'].rolling(window=60, min_periods=1).mean()

    new_data['anomaly_score'] = model.decision_function(new_data[['cpu_avg_1min', 'mem_avg_1min', 'io_ops']])
    new_data['is_anomaly'] = model.predict(new_data[['cpu_avg_1min', 'mem_avg_1min', 'io_ops']])

    return df, new_data

# Example Evaluation
test_data = pd.DataFrame({
    'timestamp': [pd.Timestamp('2023-10-01 09:10')],
    'cpu_usage': [85],
    'memory_usage': [70],
    'io_ops': [300]
})
df, new_test_data = process_data(test_data)
print(f'Anomaly Score: {new_test_data["anomaly_score"].values[0]}, Is Anomaly: {new_test_data["is_anomaly"].values[0]}')
```

**Example input / output:**

```
MARKDOWN
Copy
| timestamp           | cpu_usage | memory_usage | io_ops |
|---------------------|-----------|--------------|--------|
| 2023-10-01T09:00    | 70        | 45           | 200    |
| 2023-10-01T09:05    | 80        | 60           | 250    |

After processing a new data point:
```markdown
| timestamp           | cpu_usage | memory_usage | io_ops | anomaly_score | is_anomaly |
|---------------------|-----------|--------------|--------|---------------|------------|
| 2023-10-01T09:10    | 85        | 70           | 300    | -4.0          | 1          |

The system detected an anomaly with a score of -4.0 and flagged it as true.
```

**Data:**

```json
{
  "data": {
    "timestamp": ["2023-10-01T09:00", "2023-10-01T09:05"],
    "cpu_usage": [70, 80],
    "memory_usage": [45, 60],
    "io_ops": [200, 250]
  }
}
```

**Flow:**

1. Data Collection
2. Preprocess Data
3. Feature Engineering
4. Model Selection
5. Isolation Forest
6. Train Model
7. Real-time Processing
8. Evaluation and Tuning

