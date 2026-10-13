---
type: "practice"
difficulty: "Deep"
---

# How would you detect and alert on partition skew in production before it turns into an on-call incident during a big traffic day?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Concurrent Partition Overwrites](<../../../../02_Data_Engineering/Storage_and_Lakehouse/Partitioning_and_Bucketing/Concurrent_Partition_Overwrites.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Detecting and alerting on partition skew is critical for maintaining performance and avoiding outages, especially during high-traffic days. Partition skew can lead to inefficient data processing and potential bottlenecks. This problem involves monitoring the distribution of data across partitions and triggering alerts when the skew exceeds a certain threshold.

The core idea here is to use a combination of ETL processes, metric collection, and alerting mechanisms to proactively monitor partition skew. We'll start by defining what constitutes "partition skew" and then explore how we can implement a solution step-by-step from first principles.

**Comparison:**

```
Event Date	Skew Percentage
2023-05-15	-15.64
2023-05-20	8.79
```

**Code:**

```
Example code for detecting partition skew

import pandas as pd from datetime import datetime

def calculate_partition_skew(data, partition_column): """ Calculate the partition skew based on the distribution of data in the partition column.

CODE
Copy
:param data: DataFrame containing the data to be analyzed
:param partition_column: Column name used for partitioning
:return: A dictionary with partition names and their respective skew values
"""
# Group by partition column and calculate row counts
grouped_data = data.groupby(partition_column).size().reset_index(name='row_count')

total_rows = len(data)

# Calculate the expected number of rows per partition
expected_partition_size = total_rows / len(grouped_data)

skew_results = {}
for _, row in grouped_data.iterrows():
    partition_name = row[partition_column]
    actual_partition_size = row['row_count']

    # Skew calculation: (abs(actual - expected) / total) * 100
    skew_percentage = abs((actual_partition_size - expected_partition_size) / total_rows) * 100

    skew_results[partition_name] = skew_percentage

return skew_results

def alert_skew(partition_skew, threshold=5): """ Alert on partitions where the skew exceeds a specified threshold.

CODE
Copy
:param partition_skew: Dictionary with partition names and their respective skew values
:param threshold: The skew threshold for triggering an alert (default 5%)
:return: A list of partitions that exceed the threshold
"""
alerts = []
for partition, skew in partition_skew.items():
    if skew > threshold:
        alerts.append(partition)

return alerts
Example DataFrame

data = pd.DataFrame({ 'user_id': range(100), 'event_time': [datetime(2023, 5, i) for i in range(1, 101)], 'action': ['login', 'signup', 'purchase'] })

partition_column = 'event_time' skew_results = calculate_partition_skew(data, partition_column) alerts = alert_skew(skew_results)

alerts

CODE
Copy

```sql
-- Example SQL query to gather data for skew calculation

WITH daily_events AS (
    SELECT
        DATE(event_time) AS event_date,
        COUNT(*) AS events_count
    FROM
        user_activity
    GROUP BY
        DATE(event_time)
)

SELECT
    event_date,
    (events_count - AVG(events_count) OVER ()) / AVG(events_count) * 100.0 AS skew_percentage
FROM
    daily_events
ORDER BY
    ABS(skew_percentage) DESC;
```

**Example input / output:**

```

Input:

DataFrame with user activity data

Output:

Dictionary of partition names and their skew percentages
List of partitions that exceed the specified threshold (e.g., [2023-05-15, 2023-05-20])

Example dry-run:

PYTHON
Copy
data = pd.DataFrame({
    'user_id': range(100),
    'event_time': [datetime(2023, 5, i) for i in range(1, 101)],
    'action': ['login', 'signup', 'purchase']
})

partition_column = 'event_time'
skew_results = calculate_partition_skew(data, partition_column)
alerts = alert_skew(skew_results)

# Output: []
```

**Data:**

```json
{
  "data": [
    {"user_id": 1, "event_time": "2023-05-01T00:00:00Z", "action": "login"},
    {"user_id": 2, "event_time": "2023-05-01T00:01:00Z", "action": "signup"},
    // ... more rows
  ],
  "schema": {
    "fields": [
      {"name": "user_id", "type": "int"},
      {"name": "event_time", "type": "timestamp"},
      {"name": "action", "type": "string"}
    ]
  }
}
```

**Flow:**

1. Yes
2. Yes
3. No
4. ETL Process
5. Data Available?
6. Calculate Partition Skew
7. Skew Exceeds Threshold?
8. Alert On Call Team
9. Continue Monitoring

