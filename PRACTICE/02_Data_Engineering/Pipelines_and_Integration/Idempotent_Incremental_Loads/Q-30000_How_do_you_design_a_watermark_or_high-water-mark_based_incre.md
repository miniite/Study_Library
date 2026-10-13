---
type: "practice"
difficulty: "Deep"
---

# How do you design a watermark or high-water-mark based incremental pull from a source that only exposes an updated_at column that can go backwards?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Idempotent Incremental Loads](<../../../../02_Data_Engineering/Pipelines_and_Integration/Idempotent_Incremental_Loads/Idempotent_Incremental_Loads.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary

When designing a watermark-based incremental pipeline with a source that only exposes an updated_at timestamp but can occasionally move backwards, we need to ensure our system handles these cases correctly. The solution involves capturing watermarks, detecting and handling backward movements, and implementing a robust checkpoint mechanism.

#### Triage

The core of the question is about designing an idempotent and incremental data pipeline that can handle scenarios where the updated_at timestamp might move backwards. This requires tracking the latest known watermark (the highest updated_at value) to determine which records need to be processed, while also accounting for potential issues caused by backward timestamps.

#### First-Principles Solution
Capture Watermarks: Each time a record is pulled from the source, store its updated_at timestamp.
Detect Backward Movements: Implement logic to detect if a newer timestamp is encountered that is less than any previously seen timestamp (indicating backward movement).
Handle Backward Movements:
Skip records with timestamps that are older than the current watermark but younger than the previous watermark.
Log and handle these cases appropriately, possibly by skipping them or triggering a notification.
Checkpoints: Use a reliable checkpoint mechanism (e.g., a database table) to store the latest known watermark.
Detailed Steps
Initialization:
Initialize the pipeline with the earliest possible updated_at value from the source as the initial watermark.
Pulling Data:
Query the source for records where updated_at > last_known_watermark.
Processing Records:
For each record, compare its updated_at with the current watermark.
If the timestamp is newer (greater), update the watermark and process the record.
If the timestamp is older but younger than the previous watermark, log it as a potential backward movement and skip processing.
Checkpoint Mechanism:
Store the latest known watermark in a reliable storage system (e.g., a database table).
Handling Backward Movements:
Implement logic to handle these cases, such as skipping records or logging them for further investigation.

#### Example
Workflow
Suppose the initial watermark is 2023-10-01 12:00:00.
During a pull, a record with updated_at = 2023-10-01 14:00:00 is encountered. Update the watermark and process the record.
In another pull, a record with updated_at = 2023-10-01 13:30:00 is encountered. This timestamp is older than the current watermark but younger than the previous watermark, so log it and skip processing.

#### Closing Summary

By capturing watermarks, detecting backward movements, handling them appropriately, and maintaining a reliable checkpoint mechanism, we can ensure that our incremental pipeline operates correctly even when the updated_at column can move backwards. This approach ensures idempotency and incremental updates without reprocessing unnecessary data or missing critical changes.

#### Common Mistakes
and Follow-Ups
Not Handling Backward Movements: Failing to log and handle cases where a newer timestamp is older than the previous watermark.
Inadequate Checkpoint Mechanism: Using unreliable storage for checkpoints, leading to potential data loss or inconsistency.
Skipping Records Incorrectly: Not logging skipped records appropriately, which can lead to missed critical updates.

#### Example
Code
PYTHON
def process_records(records, current_watermark):
    processed_records = []
    skipped_records = []

    for record in records:
        updated_at = record['updated_at']
        if updated_at > current_watermark:
            # Update watermark and process the record
            current_watermark = updated_at
            processed_records.append(record)
        elif updated_at < current_watermark:
            # Log potential backward movement and skip processing
            skipped_records.append(record)

    return processed_records, skipped_records, current_watermark

# Example usage
initial_watermark = '2023-10-01 12:00:00'
records = [
    {'id': 1, 'updated_at': '2023-10-01 14:00:00'},
    {'id': 2, 'updated_at': '2023-10-01 13:30:00'}
]

processed_records, skipped_records, new_watermark = process_records(records, initial_watermark)
print(f"Processed Records: {processed_records}")
print(f"Skipped Records: {skipped_records}")
print(f"New Watermark: {new_watermark}")
Correctness + Completeness Confirmation

The provided code and solution cover the essential steps required to handle backward timestamps in an incremental pipeline.

**Code:**

```python
def process_records(records, current_watermark):
    processed_records = []
    skipped_records = []

    for record in records:
        updated_at = record['updated_at']
        if updated_at > current_watermark:
            # Update watermark and process the record
            current_watermark = updated_at
            processed_records.append(record)
        elif updated_at < current_watermark:
            # Log potential backward movement and skip processing
            skipped_records.append(record)

    return processed_records, skipped_records, current_watermark

# Example usage
initial_watermark = '2023-10-01 12:00:00'
records = [
    {'id': 1, 'updated_at': '2023-10-01 14:00:00'},
    {'id': 2, 'updated_at': '2023-10-01 13:30:00'}
]

processed_records, skipped_records, new_watermark = process_records(records, initial_watermark)
print(f"Processed Records: {processed_records}")
print(f"Skipped Records: {skipped_records}")
print(f"New Watermark: {new_watermark}")
```

**Example input / output:**

```

initial_watermark = '2023-10-01 12:00:00' records = [ {'id': 1, 'updated_at': '2023-10-01 14:00:00'}, {'id': 2, 'updated_at': '2023-10-01 13:30:00'} ]

processed_records, skipped_records, new_watermark = process_records(records, initial_watermark) print(f"Processed Records: {processed_records}") print(f"Skipped Records: {skipped_records}") print(f"New Watermark: {new_watermark}")

CODE
Copy

Output:
```plaintext
Processed Records: [{'id': 1, 'updated_at': '2023-10-01 14:00:00'}]
Skipped Records: [{'id': 2, 'updated_at': '2023-10-01 13:30:00'}]
New Watermark: 2023-10-01 14:00:00
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Initialize watermark
6. Is updated_at newer?
7. Update watermark and processrecord
8. Continue processing records
9. Is updated_at younger thanprevious watermark?
10. Log potential backwardmovement, skip processing

