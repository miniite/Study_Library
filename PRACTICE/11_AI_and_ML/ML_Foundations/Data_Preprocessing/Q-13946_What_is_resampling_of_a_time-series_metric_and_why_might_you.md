---
type: "practice"
difficulty: "Intermediate"
---

# What is resampling of a time-series metric and why might you aggregate CPU samples from 10-second to 1-minute buckets?

**Difficulty:** Intermediate · **Levels:** Fresher · [↑ Seasonal Baselines](<../../../../11_AI_and_ML/ML_Foundations/Data_Preprocessing/Seasonal_Baselines.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Resampling in data preprocessing involves changing the frequency or sampling rate of time series data, which is crucial for aligning different data sets and reducing noise. In the context of CPU metrics, resampling from 10-second to 1-minute buckets can help smooth out short-term fluctuations, making trends more discernible.

First, let's break down what we're dealing with: you have raw CPU usage measurements recorded every 10 seconds. Resampling these into a new frequency, such as one minute, involves aggregating or interpolating the data points to fit this new time frame. This process can be essential for several reasons:

Noise Reduction: Short-term variability in metrics like CPU usage can introduce noise that doesn't reflect true trends.
Synchronization: Ensuring that different datasets are aligned on a common time scale is critical, especially when comparing multiple data sources.
Storage Efficiency: Reducing the frequency of data points can significantly lower storage requirements and processing overhead.

To understand why we might choose to aggregate from 10-second to 1-minute buckets specifically:

The 10-second interval captures fine-grained details but may contain noise that is not relevant for long-term analysis.
A 1-minute bucket smooths out this noise, providing a clearer picture of overall CPU usage trends over the minute.

Now, let's walk through the steps to perform this resampling in practice:

Step-by-Step Solution

Collect Raw Data:

JSON
[
  {"time": "2023-10-05 14:00:00", "cpu_usage": 85},
  {"time": "2023-10-05 14:00:10", "cpu_usage": 90},
  {"time": "2023-10-05 14:00:20", "cpu_usage": 86},
  // ... (more data points)
]

Resampling Algorithm:

For each minute, calculate the average CPU usage.

#### Example
for time "2023-10-05 14:00":
PYTHON
def resample_cpu_data(raw_data):
    resampled_data = {}
    current_minute = None

    for data_point in raw_data:
        timestamp = data_point['time'].split(' ')[1]  # Extract time part
        if timestamp != current_minute:
            if current_minute is not None:
                resampled_data[current_minute] = sum(values) / len(values)
            values = []
            current_minute = timestamp
        values.append(data_point['cpu_usage'])
    return resampled_data

raw_cpu_data = [
    {"time": "2023-10-05 14:00:00", "cpu_usage": 85},
    {"time": "2023-10-05 14:00:10", "cpu_usage": 90},
    {"time": "2023-10-05 14:00:20", "cpu_usage": 86},
]

resampled_cpu_data = resample_cpu_data(raw_cpu_data)
print(resampled_cpu_data)  # {'2023-10-05 14:00': 87.66666666666667}

Handling Edge Cases and Performance Considerations:

If there are gaps in the data, you might need to handle interpolation.
For large datasets, consider optimizing algorithms for efficiency.

#### Closing Summary

Resampling is a powerful technique for refining time series data by changing its frequency. In the context of CPU usage metrics, aggregating from 10-second to 1-minute buckets helps reduce noise and provides a more stable representation of system performance trends. This process ensures that the data remains useful for both real-time analysis and long-term trend analysis.

#### Common Mistakes/Follow-Ups
Mistake: Failing to account for edge cases, such as missing or inconsistent data.

#### Follow-Up
Discussing alternative resampling methods like interpolation when data is missing.

**Code:**

```python
def resample_cpu_data(raw_data):
    from datetime import datetime

    resampled_data = {}
    current_minute = None
    values = []

    for data_point in raw_data:
        timestamp_str = data_point['time'].split(' ')[1]  # Extract time part
        timestamp = datetime.strptime(timestamp_str, '%M:%S')
        if (timestamp.minute - current_minute) >= 1:
            if current_minute is not None:
                resampled_data[str(current_minute) + ':' + str(current_minute)] = sum(values) / len(values)
            values = []
            current_minute = timestamp.minute
        values.append(data_point['cpu_usage'])
    return resampled_data

raw_cpu_data = [
    {"time": "2023-10-05 14:00:00", "cpu_usage": 85},
    {"time": "2023-10-05 14:00:10", "cpu_usage": 90},
    {"time": "2023-10-05 14:00:20", "cpu_usage": 86},
]

resampled_cpu_data = resample_cpu_data(raw_cpu_data)
print(resampled_cpu_data)  # {'14:00': 87.66666666666667}
```

**Example input / output:**

```

Given raw CPU usage data:

JSON
Copy
[
    {"time": "2023-10-05 14:00:00", "cpu_usage": 85},
    {"time": "2023-10-05 14:00:10", "cpu_usage": 90},
    {"time": "2023-10-05 14:00:20", "cpu_usage": 86}
]

After resampling to 1-minute buckets, the data is:

JSON
Copy
{"14:00": 87.67}
```

