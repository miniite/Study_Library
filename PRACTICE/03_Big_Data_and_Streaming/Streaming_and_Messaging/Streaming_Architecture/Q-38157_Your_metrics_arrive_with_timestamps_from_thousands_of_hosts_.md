---
type: "practice"
difficulty: "Deep"
---

# Your metrics arrive with timestamps from thousands of hosts whose clocks drift by seconds. How does clock skew corrupt event-time windowing for anomaly detection, and when do you override host time with ingestion time?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Event-time Watermarks](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Provisional_and_Final_Results/Event-time_Watermarks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with real-time streaming architectures, accurate timestamp handling is crucial for proper event-time windowing in anomaly detection systems. Clock skew between hosts introduces variability in timestamps, which can corrupt the integrity of temporal windows used to detect anomalies. This issue arises because event-time windows rely on strictly ordered and consistent timestamps.

To fully understand how clock skew affects real-time processing and what measures we might take to mitigate it, let's break down the problem step by step.

**Comparison:**

| Window Type | Impact of Clock Skew |
|-------------|----------------------|
| Tumbling    | Consistent window boundaries but may miss events or overlap windows due to skew |
| Sliding     | Flexible, can adjust for skew by overlapping windows but still prone to false positives/negatives |

**Code:**

```python
# Example of a basic window function in a stream processing framework (e.g., Apache Beam)
def process_element(element):
    timestamp = element['timestamp']
    value = element['value']

    # Apply windowing logic based on event time
    if timestamp >= start_time and timestamp < end_time:
        process_window_elements(value)
```

**Example input / output:**

| Input (Raw Data)                     | Timestamp Skew Impact   |
|--------------------------------------|------------------------|
| Host 1, Time: 2023-10-01T14:05:02Z   | Event-time window [14:05:02 - 14:06:02]  |
| Host 2, Time: 2023-10-01T14:05:03Z   | Event-time window [14:05:02 - 14:06:02] (overlaps with Host 1) |

**Data:**

```json
{
  "host1": [
    {"timestamp": "2023-10-01T14:05:02Z", "value": 10},
    {"timestamp": "2023-10-01T14:06:03Z", "value": 12}
  ],
  "host2": [
    {"timestamp": "2023-10-01T14:05:03Z", "value": 11},
    {"timestamp": "2023-10-01T14:06:04Z", "value": 13}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Input Data
4. Is Clock Skew Significant?
5. Override Host Time withIngestion Time
6. Anomaly Detection
7. Use Event Time WindowingDirectly
8. Anomaly Detection

