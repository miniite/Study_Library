---
type: "practice"
difficulty: "Deep"
---

# How would you label a window of metrics as anomalous versus normal using historical incident timestamps?

**Difficulty:** Deep · **Levels:** Fresher · [↑ Scaling with Contaminated Data](<../../../../11_AI_and_ML/ML_Foundations/Data_Preprocessing/Scaling_with_Contaminated_Data.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To determine whether a window of metrics is labeled as anomalous or normal based on historical incident timestamps, we need to analyze the presence and frequency of incidents within the given time frame. The core idea here involves understanding the historical context and applying statistical methods to detect anomalies.

The question is essentially probing your ability to perform data preprocessing and cleaning, specifically focusing on identifying patterns that deviate from expected behavior in a time series dataset. We'll need to assume some historical incident timestamps are available and map them against the metrics we're analyzing. Here’s how you can approach this step-by-step:

Data Collection: Gather historical incident timestamps for reference.
Window Definition: Define what constitutes a "window" of metrics, i.e., the time period over which the anomaly detection will be performed.
Frequency Analysis: Count incidents within each window and compare against expected or typical behavior.
Threshold Setting: Establish thresholds to determine if the number of incidents is above normal levels, indicating an anomalous state.
Normalization and Standardization: Normalize the metrics data to ensure consistency across different scales.

Let’s walk through these steps in detail:

Data Collection:

Collect historical incident timestamps from a database or log files.

Window Definition:

Define the length of the time window based on business requirements (e.g., 5 minutes, 30 minutes).

Frequency Analysis:

For each window, count the number of incidents that fall within it.
Calculate the average and standard deviation of incident frequency over a historical period to establish baseline behavior.

Threshold Setting:

Set thresholds for normal vs anomalous based on statistical analysis (e.g., using Z-scores or IQR method).
Anomalies can be detected if the number of incidents in any given window significantly deviates from the mean and standard deviation calculated over historical data.

Normalization and Standardization:

Normalize the metrics to a common scale for consistent comparison across different types of metrics.
Standardize the metric values using techniques like Z-score normalization or Min-Max scaling.

Here’s an example of how you might implement this in Python:

PYTHON
import pandas as pd
from scipy.stats import zscore

# Sample data: timestamps and corresponding metrics (dummy data for illustration)
data = {
    'timestamp': ['2023-10-01 08:00', '2023-10-01 08:05', '2023-10-01 08:10', ...],
    'metric_value': [4.5, 5.2, 6.0, ...]
}

df = pd.DataFrame(data)
df['timestamp'] = pd.to_datetime(df['timestamp'])

# Define the window size (e.g., 30 minutes)
window_size_minutes = 30
window_size_seconds = window_size_minutes * 60

# Create a sliding window over the data
windows = df.groupby((df.timestamp - df.timestamp.min()).dt.total_seconds() // window_size_seconds)

# Calculate Z-scores for each metric in the windows
z_scores = windows['metric_value'].transform(zscore)

# Identify anomalies based on z-score threshold (e.g., |Z| > 3)
anomaly_threshold = 3
df['is_anomalous'] = abs(z_scores) > anomaly_threshold

# Output results
print(df[['timestamp', 'metric_value', 'is_anomalous']])

**Comparison:**

```
timestamp	metric_value	is_anomalous
2023-10-01 08:00	4.5	False
2023-10-01 08:05	5.2	False
...	...	...
2023-10-01 09:40	7.5	True
```

**Code:**

```python
import pandas as pd
from scipy.stats import zscore

# Sample data: timestamps and corresponding metrics (dummy data for illustration)
data = {
    'timestamp': ['2023-10-01 08:00', '2023-10-01 08:05', '2023-10-01 08:10', ...],
    'metric_value': [4.5, 5.2, 6.0, ...]
}

df = pd.DataFrame(data)
df['timestamp'] = pd.to_datetime(df['timestamp'])

# Define the window size (e.g., 30 minutes)
window_size_minutes = 30
window_size_seconds = window_size_minutes * 60

# Create a sliding window over the data
windows = df.groupby((df.timestamp - df.timestamp.min()).dt.total_seconds() // window_size_seconds)

# Calculate Z-scores for each metric in the windows
z_scores = windows['metric_value'].transform(zscore)

# Identify anomalies based on z-score threshold (e.g., |Z| > 3)
anomaly_threshold = 3
df['is_anomalous'] = abs(z_scores) > anomaly_threshold

# Output results
print(df[['timestamp', 'metric_value', 'is_anomalous']])
```

**Example input / output:**

```text
# Input DataFrame:
df = pd.DataFrame({
    'timestamp': ['2023-10-01 08:00', '2023-10-01 08:05', '2023-10-01 08:10', ...],
    'metric_value': [4.5, 5.2, 6.0, ...]
})

# Output DataFrame after applying the anomaly detection algorithm:
df[['timestamp', 'metric_value', 'is_anomalous']]
```

**Data:**

```json
{
    "data": [
        {"timestamp": "2023-10-01 08:00", "metric_value": 4.5},
        {"timestamp": "2023-10-01 08:05", "metric_value": 5.2},
        {"timestamp": "2023-10-01 08:10", "metric_value": 6.0},
        ...,
        {"timestamp": "2023-10-01 09:40", "metric_value": 7.5}
    ]
}
```

**Flow:**

1. Data Collection
2. Define Window
3. Normalize Metrics
4. Frequency Analysis
5. Set Thresholds
6. Identify Anomalies
7. Output Results

