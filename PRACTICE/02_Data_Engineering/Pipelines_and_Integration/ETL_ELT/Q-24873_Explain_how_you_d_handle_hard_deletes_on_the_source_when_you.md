---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd handle hard deletes on the source when your CDC stream only reliably captures inserts and updates.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Handling Source Changes and Spikes](<../../../../02_Data_Engineering/Pipelines_and_Integration/ETL_ELT/Handling_Source_Changes_and_Spikes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with hard deletes in the source system but having a Change Data Capture (CDC) stream that only tracks insertions and updates, we need to devise an effective strategy to ensure data integrity. This involves understanding how to infer deletions from changes over time and potentially using auxiliary sources or techniques like watermarking. The core idea is to reconcile the source of truth with the CDC stream to detect any missing deletions.

First, let's break down what the interviewer might be probing:

Understanding of Data Integrity: How well you can ensure that all data in the target system accurately reflects the state of the source system.
Complexity Management: Managing the added complexity due to the missing deletion tracking in CDC streams.
Risk Assessment and Mitigation: Evaluating potential risks and proposing robust mitigation strategies.

We'll assume the following:

The CDC stream captures all insertions and updates but lacks hard deletes.
The source system occasionally performs hard deletions that are not reflected in the CDC stream.
Our goal is to maintain data integrity in the target system by detecting and applying these missing deletions.
Step-by-Step Solution
1. Understand the Data Flow

The primary challenge here is reconciling the CDC stream with the source system's hard deletes. To do this, we can use a combination of watermarking, auxiliary sources, and change detection techniques.

2. Watermarking

Watermarking helps in tracking when changes were made in the source system. By associating a timestamp (watermark) with each CDC event, we can detect any data that should have been deleted but wasn't.

Source System
CDC Stream
Watermarking System
ETL/ELT Process
Target System
3. Change Detection and Inference

To detect missing deletions, we can periodically scan the source system's transaction logs or audit trails to identify any hard deletes that aren't captured in the CDC stream.

A
B
C
D
E

Here’s a high-level approach:

Periodic Scans: Set up periodic scans of the source system's transaction logs or audit trails.
Watermark Comparison: Compare these timestamps with those in the CDC stream to identify missing deletions.
4. Auxiliary Sources

If the source system has an auxiliary data store (like a database) that tracks all operations, we can use this as a reference to detect hard deletes not captured by the CDC stream.

A
Auxiliary Data Store
CDC Stream
ETL/ELT Process
5. ETL/ELT Process

The ETL/ELT process needs to be designed to handle these inferred deletions:

Identify Missing Deletions: Use the watermarking and auxiliary sources to identify any missing deletions.
Apply Inferred Deletions: Apply these inferred deletions to the target system.
PYTHON
# Example ETL/ELT Process Code Snippet

def reconcile_cdc_stream(cdc_data, source_system_logs):
    # Initialize state with CDC data
    current_state = cdc_data

    # Iterate over source system logs and apply missing deletions
    for log_entry in source_system_logs:
        if is_deletion(log_entry):
            entity_id = get_entity_id(log_entry)

            # Apply inferred deletion to the target system's state
            delete_from_target(entity_id, current_state)

def is_deletion(log_entry):
    # Logic to determine if a log entry represents a deletion

def get_entity_id(log_entry):
    # Extract entity ID from log entry

def delete_from_target(entity_id, current_state):
    # Implement logic to delete the entity from the target system's state
6. Monitoring and Validation

Finally, set up monitoring and validation mechanisms to ensure that inferred deletions are applied correctly and that data integrity is maintained.

#### Closing Summary

Handling hard deletes in a source system when only CDC inserts and updates are tracked requires a robust reconciliation strategy involving watermarking, periodic scans of transaction logs, and the use of auxiliary sources. The ETL/ELT process must be designed to apply inferred deletions accurately to maintain data integrity.

#### Common Mistakes
and Follow-Ups
Missing Watermarking: Failing to capture timestamps for CDC events can lead to missed detections.
Inadequate Logging: Not having proper transaction logs or audit trails in the source system can make it difficult to detect missing deletions.
Overly Aggressive Deletions: Applying inferred deletions without thorough validation could result in incorrect data deletion.

**Comparison:**

```
CDC Data	Source System Logs
id: 1	id: 3
timestamp: 2023-10-01T14:00:00Z	timestamp: 2023-10-02T16:00:00Z, type: delete
action: insert	id: 4
id: 2	timestamp: 2023-10-03T17:00:00Z, type: insert
timestamp: 2023-10-01T15:00:00Z
action: update
```

**Code:**

```python
# Example ETL/ELT Process Code Snippet

def reconcile_cdc_stream(cdc_data, source_system_logs):
    current_state = cdc_data

    for log_entry in source_system_logs:
        if is_deletion(log_entry):
            entity_id = get_entity_id(log_entry)

            delete_from_target(entity_id, current_state)

def is_deletion(log_entry):
    # Logic to determine if a log entry represents a deletion
    pass

def get_entity_id(log_entry):
    # Extract entity ID from log entry
    pass

def delete_from_target(entity_id, current_state):
    # Implement logic to delete the entity from the target system's state
    pass
```

**Example input / output:**

```text
# Example input and output

cdc_data = [
    {"id": 1, "timestamp": "2023-10-01T14:00:00Z", "action": "insert"},
    {"id": 2, "timestamp": "2023-10-01T15:00:00Z", "action": "update"}
]

source_system_logs = [
    {"id": 3, "timestamp": "2023-10-02T16:00:00Z", "type": "delete"},
    {"id": 4, "timestamp": "2023-10-03T17:00:00Z", "type": "insert"}
]

reconcile_cdc_stream(cdc_data, source_system_logs)
```

**Data:**

```json
{
  "cdc_data": [
    {"id": 1, "timestamp": "2023-10-01T14:00:00Z", "action": "insert"},
    {"id": 2, "timestamp": "2023-10-01T15:00:00Z", "action": "update"}
  ],
  "source_system_logs": [
    {"id": 3, "timestamp": "2023-10-02T16:00:00Z", "type": "delete"},
    {"id": 4, "timestamp": "2023-10-03T17:00:00Z", "type": "insert"}
  ]
}
```

**Flow:**

1. Source System
2. CDC Stream
3. Watermarking System
4. ETL/ELT Process
5. Target System

