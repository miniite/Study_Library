---
type: "practice"
difficulty: "Deep"
---

# What is your approach when the same anomaly fires across 5000 hosts and you need to deduplicate to one incident?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Dynamic vs Static Alerting](<../../../../11_AI_and_ML/Applied_ML/Anomaly_Detection/Dynamic_vs_Static_Alerting.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with anomalies in a distributed system, especially across multiple hosts, the challenge often lies in efficiently deduplicating similar events into fewer, more actionable incidents. In this scenario, where 5000 hosts report the same anomaly, we need to create a robust solution that not only identifies and groups these anomalies but also ensures minimal false positives while maintaining high accuracy.

To tackle this problem, we must first understand what "the same" anomaly means in terms of time series data. Typically, it involves comparing patterns or values across different time series datasets. The key idea is to use a combination of statistical methods and machine learning models to detect anomalies and then apply deduplication techniques based on the similarity score between these anomalies.

**Comparison:**

```
Grouped Anomalies	Host IDs
[1, 2]	[3, 4, 5]
[6, 7]	[8, 9]
```

**Code:**

```python
import numpy as np
from scipy.stats import zscore

def calculate_similarity(ts1, ts2):
    # Calculate Z-score for each time series
    z1 = zscore(ts1)
    z2 = zscore(ts2)

    # Compute the correlation between the two time series
    similarity_score = np.corrcoef(z1, z2)[0, 1]
    return similarity_score

def group_anomalies(anomalies):
    """
    Anomalies is a list of (timestamp, host_id) tuples.
    """
    grouped_anomalies = {}

    for anomaly in anomalies:
        timestamp, host_id = anomaly
        key = str(timestamp)

        if key not in grouped_anomalies:
            grouped_anomalies[key] = [(timestamp, host_id)]
        else:
            existing_hosts = [h[1] for h in grouped_anomalies[key]]

            # Deduplicate hosts with high similarity score
            for _, existing_host in grouped_anomalities[key]:
                if calculate_similarity(anomalies[existing_host][0], anomalies[host_id][0]) > 0.8:
                    break
            else:
                grouped_anomalies[key].append((timestamp, host_id))

    return list(grouped_anomalities.values())
```

**Example input / output:**

```

Given the input data:

JSON
Copy
[
    {"timestamp": "2023-10-01T12:00Z", "host_id": 1, "value": 98},
    {"timestamp": "2023-10-01T12:05Z", "host_id": 2, "value": 97},
    // ... 4998 more entries
]

After processing with the group_anomalies function, we get:

JSON
Copy
[
    [
        {"timestamp": "2023-10-01T12:00Z", "host_id": 1},
        {"timestamp": "2023-10-01T12:05Z", "host_id": 2}
    ],
    // ... other groups
]
```

**Data:**

```json
[
    {"timestamp": "2023-10-01T12:00Z", "host_id": 1, "value": 98},
    {"timestamp": "2023-10-01T12:05Z", "host_id": 2, "value": 97},
    // ... 4998 more entries
]
```

**Flow:**

1. Yes
2. No
3. Anomaly Detection Phase
4. Is Anomaly Similar?
5. Deduplicate Incident
6. Create New Incident

