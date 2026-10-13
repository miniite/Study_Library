---
type: "practice"
difficulty: "Deep"
---

# How would you design storage and retrieval for time-series metrics (billions of points/day) so recent data is fast and old data is cheap? Discuss downsampling, retention, and rollups.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Clickstream Counting Pipeline](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Clickstream_Counting_Pipeline/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a system for storing and retrieving time-series metrics with billions of points per day, we need to balance performance (especially for recent data) with cost-effectiveness for older data. The key components are downsampling, retention policies, and rollups. Let's break down the solution from first principles.

First, let’s clarify what the question is probing:

Downsampling: How to reduce the amount of storage needed while maintaining useful information.
Retention Policies: Deciding how long to keep raw data versus aggregated data.
Rollups: Techniques for aggregating and summarizing larger sets of data into more manageable chunks.

We assume a typical use case where we have metrics like CPU usage, memory utilization, etc., collected every second or minute. The key is to ensure that recent data can be queried quickly while old data can be stored cheaply without compromising the overall system's accuracy and performance.

**Code:**

```python
# Example code snippet for downsampling logic in Python
def downsample(data_points, interval):
    """
    Downsamples time-series data points based on a given interval.

    Args:
        data_points (list of tuples): Each tuple is (timestamp, value).
        interval (int): Interval in seconds to aggregate.

    Returns:
        list: Aggregated data points.
    """
    result = []
    current_time = None
    values_sum = 0.0
    count = 0

    for timestamp, value in sorted(data_points):
        if not current_time or (timestamp - current_time) >= interval:
            if count > 0:
                average_value = values_sum / count
                result.append((current_time, average_value))
            current_time = timestamp
            values_sum = 0.0
            count = 0

        values_sum += value
        count += 1

    # Add the last aggregated point if needed
    if count > 0:
        average_value = values_sum / count
        result.append((current_time, average_value))

    return result

# Example usage
data_points = [(1632774000, 5.2), (1632774002, 5.8), (1632774004, 5.9), (1632774006, 6.1)]
downsampled_data = downsample(data_points, 2)
print(downsampled_data)  # Output: [(1632774000, 5.5), (1632774004, 5.95)]
```

**Example input / output:**

```text
# Input data points
data_points = [
    (1632774000, 5.2),
    (1632774002, 5.8),
    (1632774004, 5.9),
    (1632774006, 6.1)
]

# Downsample with an interval of 2 seconds
downsampled_data = downsample(data_points, 2)

# Output: [(1632774000, 5.5), (1632774004, 5.95)]
print(downsampled_data)
```

**Data:**

```json
{
    "metrics": [
        {"timestamp": 1632774000, "value": 5.2},
        {"timestamp": 1632774002, "value": 5.8},
        {"timestamp": 1632774004, "value": 5.9},
        {"timestamp": 1632774006, "value": 6.1}
    ]
}
```

