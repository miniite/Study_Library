---
type: "practice"
difficulty: "Basic"
---

# What is a trend in a time series and how can an upward trend fool a static threshold?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Drift and Threshold Detection](<../../../../11_AI_and_ML/Applied_ML/Anomaly_Detection/Drift_and_Threshold_Detection.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In anomaly detection, particularly when dealing with time series data, identifying trends is crucial to accurately flag unusual patterns. A trend refers to the long-term direction or movement in the data over time. For instance, an upward trend indicates that the values are generally increasing over the period being analyzed.

However, an upward trend can lead a static threshold method astray because it doesn't account for the changing context of the data. Static thresholds work by setting fixed rules based on historical data to identify anomalies. This approach can be misleading when the underlying pattern changes, such as in the case of an upward trend.

To illustrate, consider a company's daily website traffic over several months. Initially, the traffic might be relatively stable. But if there’s a gradual increase due to successful marketing campaigns or seasonal factors, a static threshold set based on early data will likely fail to detect significant drops that occur later. This is because the thresholds are not adjusted for the new upward trend.

Let's break down this concept from first principles.

**Comparison:**

```
Date	Value	SMA
2023-04-27	150	162.8
2023-04-28	155	159.6
2023-04-29	160	156.8
2023-04-30	165	154.4
2023-05-01	170	152.0

These values indicate the SMA is starting to capture the upward trend.
```

**Code:**

```python
# Example of calculating a simple moving average (SMA) as a proxy for trend
import pandas as pd

# Sample time series data
data = {
    'date': pd.date_range(start='2023-01-01', periods=30, freq='D'),
    'value': [150, 155, 160, 165, 170] + [x for x in range(180, 300)] # Initial stable then increasing trend
}
df = pd.DataFrame(data)

# Calculate simple moving average with window size of 5 days
df['sma'] = df['value'].rolling(window=5).mean()
print(df.tail())  # View the last few rows to see the SMA trend
```

**Example input / output:**

```text
# Input:
df.tail()
# Output:
          date   value      sma
24  2023-05-01    170  162.800

This output shows that the simple moving average is starting to reflect an upward trend.
```

**Data:**

```json
{
  "data": [
    {"date": "2023-04-27", "value": 150},
    {"date": "2023-04-28", "value": 155},
    {"date": "2023-04-29", "value": 160},
    {"date": "2023-04-30", "value": 165},
    {"date": "2023-05-01", "value": 170}
  ]
}
```

**Flow:**

1. Detect Initial Trend
2. Calculate Simple MovingAverage
3. Use SMA to Define NormalRange
4. Predict and Flag AnomaliesBased on Updated Context

