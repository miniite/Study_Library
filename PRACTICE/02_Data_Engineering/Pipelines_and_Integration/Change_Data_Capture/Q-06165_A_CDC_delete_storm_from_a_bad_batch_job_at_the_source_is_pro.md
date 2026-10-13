---
type: "practice"
difficulty: "Deep"
---

# A CDC delete storm from a bad batch job at the source is propagating to the warehouse: how would you detect it and add a safety valve to stop cascade damage?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ CDC Fundamentals and Ordering](<../../../../02_Data_Engineering/Pipelines_and_Integration/Change_Data_Capture/CDC_Fundamentals_and_Ordering.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: To address a CDC delete storm from a bad batch job, we need to implement a mechanism that detects anomalous deletions and stops further propagation before causing extensive damage in the warehouse. The solution involves monitoring change events, setting up anomaly detection rules, and implementing a rollback strategy.

To tackle this issue at the source:

Monitor Change Events: Capture all delete operations from the bad batch job.
Detect Anomalies: Use statistical or machine learning models to identify unusual patterns of deletions that deviate significantly from normal behavior.
Implement Rollback Mechanism: Create a process to revert problematic changes before they reach the warehouse.

#### Triage

The question is about detecting and mitigating the impact of a CDC delete storm, which can be caused by a faulty batch job. We need to ensure that we can identify such storms early and prevent cascading deletions in downstream systems.

#### Assumptions
include:
We have access to logs or audit trails from the source system.
The warehouse has a robust CDC pipeline with proper event tracking.

#### First Principles Solution

Monitoring Change Events:

Capture all delete operations from the bad batch job using a CDC tool like Debezium, Fivetran, or custom scripts. Log these events in a staging area for further analysis.

Setting Up Anomaly Detection Rules:

Define thresholds and patterns that indicate normal versus abnormal behavior. For example:
Frequent deletions over a short period.
Deleting large volumes of data at once.
Deletions occurring during non-business hours or outside expected times.
Use statistical methods (e.g., mean, standard deviation) or machine learning models to classify events as normal or anomalous. Models can be trained on historical data where normal behavior is known.

Implementing Rollback Mechanism:

Develop a mechanism to automatically revert detected anomalies before they reach the warehouse.
Integrate with CDC tools to apply rollbacks directly at the source system if an anomaly is detected.

Monitoring and Alerting:

Set up real-time monitoring to alert stakeholders when anomalies are detected, allowing for immediate action.
Automate remediation steps as part of a failover process.

#### Closing Summary
+ Common Mistakes/Follow-Ups:

The key is to have a robust monitoring system that can quickly identify and mitigate issues before they cause significant damage. Implementing a rollback mechanism ensures that even if the bad batch job continues, it won’t propagate deletions further.

#### Common mistakes
include:
Not having a proper monitoring setup in place, leading to delayed detection of issues.
Over-relying on manual interventions instead of automated mechanisms.
Failing to test and validate anomaly detection models thoroughly.

#### Follow-Ups

How would you ensure that the rollback mechanism doesn’t cause new anomalies?
What measures would you take to prevent future CDC storms?
Can you provide an example of how you might implement this in a real-world scenario?

**Comparison:**

```
Timestamp	Entity ID	Action
2023-10-01	1	insert
2023-10-01	2	delete
2023-10-01	3	update
Timestamp	Entity ID	Action
2023-10-01	1	insert
2023-10-01	3	update

Note: The 'delete' operation for entity ID 2 was rolled back due to anomaly detection.
```

**Code:**

```python
# Example Python script for detecting anomalies and rolling back deletions

import pandas as pd
from statsmodels.stats import weightstats

def detect_anomalies(data, threshold=2):
    # Assume `data` is a DataFrame with columns 'timestamp', 'entity_id', 'action'
    mean_deletions = data[data['action'] == 'delete'].groupby('timestamp').count()['entity_id'].mean()
    std_dev = data[data['action'] == 'delete'].groupby('timestamp').count()['entity_id'].std()

    # Flag anomalies based on standard deviation from the mean
    anomalies = (data.groupby('timestamp').count()['entity_id'] - mean_deletions).abs() > threshold * std_dev

    return anomalies

def roll_back_anomalies(staging_area, warehouse):
    # Assume `staging_area` and `warehouse` are DataFrames with CDC data
    for timestamp in staging_area[staging_area['action'] == 'delete'].groupby('timestamp').apply(lambda x: detect_anomalies(x)):
        if timestamp:
            # Rollback logic here
            print(f"Rolling back deletions at {timestamp}")
```

**Example input / output:**

```text
Input:
Staging Area (anomalies detected for 'delete' actions):
Timestamp: 2023-10-01T08:59:59, Anomaly Detected: True

Output:
Rolling back deletions at 2023-10-01T08:59:59
```

**Data:**

```json
{
  "staging_area": {
    "entities": [
      {"entity_id": 1, "action": "insert", "timestamp": "2023-10-01T09:00:00"},
      {"entity_id": 2, "action": "delete", "timestamp": "2023-10-01T08:59:59"},
      {"entity_id": 3, "action": "update", "timestamp": "2023-10-01T09:00:00"}
    ]
  },
  "warehouse": {
    "entities": [
      {"entity_id": 1, "action": "insert", "timestamp": "2023-10-01T09:00:00"},
      {"entity_id": 2, "action": "delete", "timestamp": "2023-10-01T08:59:59"},
      {"entity_id": 3, "action": "update", "timestamp": "2023-10-01T09:00:00"}
    ]
  }
}
```

**Flow:**

1. Yes
2. No
3. Monitor Change Events
4. Detect Anomalies?
5. Raise Alert, Rollback
6. Continue Processing
7. Notify Stakeholders,Implement RollbackMechanism

