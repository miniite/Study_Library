---
type: "practice"
difficulty: "Intermediate"
---

# How would you detect a contextual anomaly where 90 percent CPU is fine at peak but abnormal at 3am?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Dynamic vs Static Alerting](<../../../../11_AI_and_ML/Applied_ML/Anomaly_Detection/Dynamic_vs_Static_Alerting.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the scenario of detecting a contextual anomaly in time series data, such as identifying when 90% CPU usage is normal during peak hours but unusual at 3 AM, we need to understand that traditional threshold-based methods may not suffice due to the context-dependency. The core idea is to develop a model that can adapt its thresholds based on historical patterns and time-of-day factors.

The question primarily tests your ability to:

Design a system capable of handling contextual variations.
Understand how to incorporate temporal dependencies in anomaly detection models.

#### Cons
ider different strategies for defining "normal" behavior over time.

#### Assumptions
include that we have access to historical CPU usage data and that the system can process this data efficiently. The solution will involve creating a model that learns from past patterns, adjusting its thresholds based on the current time of day.

Step-by-Step Solution
Data Collection & Preprocessing: Gather historical CPU usage data along with corresponding timestamps.
Feature Engineering: Extract relevant features such as the hour of the day, day of the week, etc., to capture temporal dependencies.
Model Selection: Choose an appropriate model that can handle time series data and context-dependency, such as a combination of autoregressive models (e.g., ARIMA) or machine learning algorithms like Random Forests or LSTM networks.
Training & Validation: Train the model using historical data, validate its performance against known anomalies or expected normal conditions.
Threshold Adjustment: Dynamically adjust thresholds based on the time of day and recent patterns to detect contextual anomalies.

Let's break down each step in detail:

Data Collection & Preprocessing:

PYTHON
import pandas as pd

# Sample data schema
cpu_data = {
    'timestamp': ['2023-10-04 15:00', '2023-10-04 21:00', '2023-10-05 03:00'],
    'cpu_usage': [87, 93, 90]
}

df = pd.DataFrame(cpu_data)
df['timestamp'] = pd.to_datetime(df['timestamp'])
df.set_index('timestamp', inplace=True)

Feature Engineering:

PYTHON
import numpy as np

def add_time_features(data):
    data['hour_of_day'] = data.index.hour
    data['day_of_week'] = data.index.dayofweek
    return data

df_enhanced = add_time_features(df)

Model Selection & Training:

PYTHON
from sklearn.model_selection import train_test_split
from sklearn.ensemble import RandomForestRegressor

X = df_enhanced[['hour_of_day', 'day_of_week']]
y = df_enhanced['cpu_usage']

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

model = RandomForestRegressor()
model.fit(X_train, y_train)

Threshold Adjustment:

PYTHON
def adjust_threshold(model, current_time):
    features = np.array([[current_time.hour_of_day, current_time.day_of_week]])
    predicted_usage = model.predict(features)[0]

    if predicted_usage > 95:  # Example threshold
        return 'anomaly'
    else:
        return 'normal'

# Example usage
time_now = df_enhanced.index[-1] + pd.Timedelta(hours=2)  # Simulate future time
print(adjust_threshold(model, time_now))

#### Closing Summary
& Common Mistakes

This approach ensures that the anomaly detection system can adapt to context-specific patterns. A common mistake is using static thresholds without considering temporal dependencies, which might lead to false positives or negatives. Additionally, failing to validate the model thoroughly against known anomalies and normal conditions can result in suboptimal performance.

**Comparison:**

```
Timestamp	CPU Usage	Hour of Day	Day of Week
2023-10-04 15:00	87	15	4
2023-10-04 21:00	93	21	4
2023-10-05 03:00	90	3	5
```

**Code:**

```python
import pandas as pd

# Sample data schema
cpu_data = {
    'timestamp': ['2023-10-04 15:00', '2023-10-04 21:00', '2023-10-05 03:00'],
    'cpu_usage': [87, 93, 90]
}
df = pd.DataFrame(cpu_data)
df['timestamp'] = pd.to_datetime(df['timestamp'])
df.set_index('timestamp', inplace=True)

def add_time_features(data):
    data['hour_of_day'] = data.index.hour
    data['day_of_week'] = data.index.dayofweek
    return data

df_enhanced = add_time_features(df)

from sklearn.model_selection import train_test_split
from sklearn.ensemble import RandomForestRegressor

X = df_enhanced[['hour_of_day', 'day_of_week']]
y = df_enhanced['cpu_usage']

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

model = RandomForestRegressor()
model.fit(X_train, y_train)

def adjust_threshold(model, current_time):
    features = np.array([[current_time.hour_of_day, current_time.day_of_week]])
    predicted_usage = model.predict(features)[0]

    if predicted_usage > 95:  # Example threshold
        return 'anomaly'
    else:
        return 'normal'

# Example usage
time_now = df_enhanced.index[-1] + pd.Timedelta(hours=2)  # Simulate future time
print(adjust_threshold(model, time_now))
```

**Example input / output:**

```text
# Example input and output
df_enhanced = pd.DataFrame({
    'timestamp': ['2023-10-04 15:00', '2023-10-04 21:00', '2023-10-05 03:00'],
    'cpu_usage': [87, 93, 90],
    'hour_of_day': [15, 21, 3],
    'day_of_week': [4, 4, 5]
})

model = RandomForestRegressor()
model.fit(df_enhanced[['hour_of_day', 'day_of_week']], df_enhanced['cpu_usage'])

time_now = pd.Timestamp('2023-10-05 03:00') + pd.Timedelta(hours=2)
print(adjust_threshold(model, time_now))  # Output: anomaly
```

**Data:**

```json
{
    "cpu_data": {
        "timestamp": ["2023-10-04 15:00", "2023-10-04 21:00", "2023-10-05 03:00"],
        "cpu_usage": [87, 93, 90]
    }
}
```

**Flow:**

1. Yes
2. Data Collection &Preprocessing
3. Feature Engineering?
4. Model Selection
5. Training Model
6. Threshold Adjustment Basedon Time
7. Anomaly Detection

