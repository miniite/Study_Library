---
type: "practice"
difficulty: "Deep"
---

# How would you architect a cost-attribution and anomaly-detection layer on the lake's own spend so a runaway ingestion source is caught in hours not at month-end?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Hot vs Cold Telemetry](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Observability_Data_Lake/Hot_vs_Cold_Telemetry.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of catching a potential cost overrun early, we need to architect a system that continuously monitors and analyzes the data being ingested into an Observability Data Lake, with a focus on both cost attribution and anomaly detection. The core idea is to implement real-time monitoring and alerting mechanisms to detect unusual spending patterns as they emerge.

This solution involves several key components: data collection, processing pipelines, storage, real-time analytics, and alerting systems. We'll break down the architecture from first principles and discuss each step in detail.

**Comparison:**

| Timestamp       | Ingestion Cost ($) | Anomaly Detected |
|-----------------|-------------------|------------------|
| 2023-10-01T12:00Z | 0.05              | false            |
| 2023-10-01T12:15Z | 0.06              | false            |
| 2023-10-01T12:30Z | 0.07              | true             |

**Code:**

```python
# Example of a simple anomaly detection function using statistical methods
import numpy as np

def detect_anomaly(values):
    mean = np.mean(values)
    std_dev = np.std(values)

    for value in values:
        z_score = (value - mean) / std_dev
        if abs(z_score) > 3:  # Threshold for anomaly detection
            return True, value
    return False, None
```

**Example input / output:**

```

Input:

Cost metrics are collected every 15 minutes from the data ingestion layer. Output:
Anomaly detection results stored in a time-series database for real-time monitoring and alerting.

Example dry-run:

Request (first 30 minutes of cost metrics):

CODE
Copy
2023-10-01T12:00Z -> $0.05
2023-10-01T12:15Z -> $0.06
2023-10-01T12:30Z -> $0.07

Response (anomaly detection results):

CODE
Copy
2023-10-01T12:00Z -> false
2023-10-01T12:15Z -> false
2023-10-01T12:30Z -> true (anomaly detected)
```

**Data:**

```json
{
  "cost_metrics": {
    "timestamp": ["2023-10-01T12:00Z", "2023-10-01T12:15Z", ...],
    "ingestion_cost": [0.05, 0.06, ...],
    "anomaly_detected": [false, true, ...]
  }
}
```

**Flow:**

1. No
2. Yes
3. Data Ingestion
4. Is Cost Spike Detected?
5. Record Cost Metrics
6. Trigger Alert
7. Real-time Monitoring
8. Log Anomaly

