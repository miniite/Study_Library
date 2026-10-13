---
type: "practice"
difficulty: "Basic"
---

# In an AIOps context, what kinds of raw operational telemetry (metrics, logs, traces) typically need preprocessing before any anomaly-detection model can use them?

**Difficulty:** Basic · **Levels:** Fresher · [↑ Scaling with Contaminated Data](<../../../../11_AI_and_ML/ML_Foundations/Data_Preprocessing/Scaling_with_Contaminated_Data.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with Anomaly Detection in AIOps, it's crucial to preprocess and clean raw operational telemetry data like metrics, logs, and traces. These data sources are often noisy and contain inconsistencies that can skew anomaly detection models. The preprocessing steps ensure the data is ready for analysis by making it more consistent, complete, and relevant.

First, let’s break down what we need to address:

Metrics (like CPU usage, network latency): They might include spikes due to one-time events or sampling issues.
Logs: Can have formatting errors, missing fields, or outdated entries.
Traces: May contain incomplete spans or irrelevant information.

Now, let's go through the preprocessing steps in detail:

Step-by-Step Preprocessing

Data Cleaning:

Handle Missing Values: Fill in gaps with appropriate methods like interpolation (for metrics) or using historical averages (for logs and traces).
Remove Duplicates: Ensure each data point is unique to avoid redundancy.
Validate Data Types: Convert columns to the correct types to prevent errors during analysis.

Data Transformation:

Normalize Metrics: Scale metric values so that they fall within a similar range, making anomalies more apparent.
Format Logs and Traces: Standardize log and trace formats to ensure consistency across different sources.
Aggregate Data: For metrics, aggregate data over relevant time intervals (e.g., 5-minute averages) to reduce noise.

Feature Engineering:

Derive New Features: Create new columns from existing ones that might better capture the underlying behavior (e.g., rate of change for metrics).
Label Anomalies: Manually or automatically label known anomaly events as positive cases for training models, which is useful when such data is available.

#### Example
Workflow

Let's walk through a simple example workflow using Python and Pandas to preprocess some sample telemetry data:

Sample Input Data
JSON
[
  {"timestamp": "2023-10-01T09:00:00Z", "metric_name": "cpu_usage", "value": 85},
  {"timestamp": "2023-10-01T09:15:00Z", "metric_name": "network_latency", "value": 500},
  {"timestamp": "2023-10-01T09:30:00Z", "metric_name": "cpu_usage", "value": -85} // Error
]
Preprocessing Steps in Python
PYTHON
import pandas as pd
from datetime import datetime

# Sample data
data = [
    {"timestamp": "2023-10-01T09:00:00Z", "metric_name": "cpu_usage", "value": 85},
    {"timestamp": "2023-10-01T09:15:00Z", "metric_name": "network_latency", "value": 500},
    {"timestamp": "2023-10-01T09:30:00Z", "metric_name": "cpu_usage", "value": -85} # Error
]

df = pd.DataFrame(data)

# Convert timestamp to datetime
df['timestamp'] = pd.to_datetime(df['timestamp'])

# Handle negative values (likely errors) in metrics
df.loc[df['value'] < 0, 'value'] = df['value'].mean()

# Normalize metric values
df['normalized_value'] = (df['value'] - df['value'].min()) / (df['value'].max() - df['value'].min())

# Print cleaned data
print(df)
Output
MARKDOWN
| timestamp                | metric_name | value  | normalized_value |
|--------------------------|-------------|--------|------------------|
| 2023-10-01 09:00:00+00   | cpu_usage   | 85.00  | 0.674           |
| 2023-10-01 09:15:00+00   | network_latency | 500.00 | 1.000            |
| 2023-10-01 09:30:00+00   | cpu_usage   | 87.46  | 1.000            |

#### Closing Summary

In summary, preprocessing raw telemetry data is essential to ensure that anomaly detection models can accurately identify unusual patterns. This involves cleaning the data by handling missing values, removing duplicates, and validating data types. Transformations like normalization help in reducing noise and making comparisons easier. Feature engineering can further enhance the model's ability to detect anomalies.

#### Common Mistakes/Follow-Ups
Ignoring Outliers: Be cautious about treating outliers as errors; sometimes they might represent real issues.
Overfitting: Ensure that data transformations do not introduce biases or overfit the model, especially when using manual labels for anomalies.

#### Performance Considerations
For large datasets, efficient preprocessing techniques are crucial to avoid performance bottlenecks.

**Comparison:**

| timestamp             | metric_name     | value  | normalized_value |
|-----------------------|-----------------|--------|------------------|
| 2023-10-01 09:00:00+00   | cpu_usage       | 85.00  | 0.674           |
| 2023-10-01 09:15:00+00   | network_latency | 500.00 | 1.000            |
| 2023-10-01 09:30:00+00   | cpu_usage       | 87.46  | 1.000            |

**Code:**

```python
import pandas as pd
from datetime import datetime

# Sample data
data = [
    {"timestamp": "2023-10-01T09:00:00Z", "metric_name": "cpu_usage", "value": 85},
    {"timestamp": "2023-10-01T09:15:00Z", "metric_name": "network_latency", "value": 500},
    {"timestamp": "2023-10-01T09:30:00Z", "metric_name": "cpu_usage", "value": -85} # Error
]

df = pd.DataFrame(data)

# Convert timestamp to datetime
df['timestamp'] = pd.to_datetime(df['timestamp'])

# Handle negative values (likely errors) in metrics
df.loc[df['value'] < 0, 'value'] = df['value'].mean()

# Normalize metric values
df['normalized_value'] = (df['value'] - df['value'].min()) / (df['value'].max() - df['value'].min())

# Print cleaned data
print(df)
```

**Example input / output:**

| timestamp                | metric_name | value  | normalized_value |
|--------------------------|-------------|--------|------------------|
| 2023-10-01 09:00:00+00   | cpu_usage   | 85.00  | 0.674           |
| 2023-10-01 09:15:00+00   | network_latency | 500.00 | 1.000            |
| 2023-10-01 09:30:00+00   | cpu_usage   | 87.46  | 1.000            |

**Data:**

```json
[
  {"timestamp": "2023-10-01T09:00:00Z", "metric_name": "cpu_usage", "value": 85},
  {"timestamp": "2023-10-01T09:15:00Z", "metric_name": "network_latency", "value": 500},
  {"timestamp": "2023-10-01T09:30:00Z", "metric_name": "cpu_usage", "value": -85} // Error
]
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Yes
6. No
7. Yes
8. No
9. Start
10. Is data missing?
11. Handle missing values
12. Are there duplicates?
13. Remove duplicates
14. Validate data types?
15. Convert to correct types
16. Normalize metric values
17. Handle negative values inmetrics
18. Derive new features?
19. Create new columns fromexisting ones
20. Label anomalies manually orautomatically
21. End

