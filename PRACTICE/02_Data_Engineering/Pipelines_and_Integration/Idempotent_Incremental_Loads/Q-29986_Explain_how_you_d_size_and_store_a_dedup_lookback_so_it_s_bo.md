---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd size and store a dedup lookback so it's bounded but still catches realistic duplicate windows.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Idempotent Incremental Loads](<../../../../02_Data_Engineering/Pipelines_and_Integration/Idempotent_Incremental_Loads/Idempotent_Incremental_Loads.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure we have an idempotent and incremental pipeline that accurately handles duplicates with a practical lookback window, we need to size our deduplication logic carefully to balance between false positives and the risk of missing actual duplicates. The key is to determine how far back in time you want to capture potential duplicates while keeping the storage and processing costs manageable.

First, let's break down what we're solving for:

Lookback Window: The duration over which we need to check for duplicate records.
Boundedness: Ensuring our storage can handle the amount of data without growing indefinitely.
Realistic Duplicate Windows: Capturing enough history to ensure we don't miss duplicates but within practical limits.

#### Triaging the Question

The interviewer is probing into how you balance the trade-offs between memory and time, ensuring that your system can reliably detect duplicates while staying efficient. We need to make assumptions about the rate of incoming data, the acceptable delay in processing, and the frequency of potential duplicates.

#### First-Principles Solution
Estimate Data Rate: Determine how many records are processed per second or minute.
Define Lookback Duration: Decide on a practical lookback window based on business needs and technical constraints.
Deduplication Strategy:
Use a combination of hash-based checks (e.g., MD5, SHA) and exact field matches for speed and accuracy.
Storage Sizing:
Calculate the storage required for your deduplication strategy.
Ensure that the storage is managed with lifecycle policies to prevent unbounded growth.
Detailed Steps

Calculate Lookback Duration: Let’s assume we want a lookback window of 7 days, which is common in many systems.

Data Rate Analysis:

If your system processes 1 million records per day (86400 seconds/day), then the total number of records to keep for 7 days would be approximately 6.5 billion.

Storage Calculation:

Assuming each record is an average size of 1 KB, the storage requirement would be around 6.5 GB.
However, we need to consider that some records might be duplicates and won't require full storage.

Deduplication Algorithm Implementation:

Use a hash map to store hashes of incoming data entries as keys and a reference to the original record as values.
For new records, compute their hash and check if it exists in the hash map.
PYTHON
def deduplicate(record):
    hash_key = hashlib.md5(record.encode()).hexdigest()
    if hash_key not in deduplication_map:
        deduplication_map[hash_key] = record
        return True  # New record, add to storage and send to processing pipeline
    else:
        return False  # Duplicate detected, no action needed

#### Closing Summary

To size a deduplication lookback window for an idempotent and incremental pipeline, we need to balance the trade-offs between false positives and missed duplicates. By estimating data rates, defining practical lookback durations, and implementing efficient storage strategies, we can ensure that our system is both effective and cost-optimized.

#### Common Mistakes/Follow-Ups
Ignoring Data Rate: Failing to consider how fast data comes in can lead to significant storage issues.
Inadequate Lookback Duration: Setting too short a lookback window might miss actual duplicates, while setting it too long can result in unnecessary storage costs.
Lack of Lifecycle Management: Not having policies for cleaning up old data can cause storage to grow indefinitely.

**Comparison:**

```
Record ID	Value
1	record1
2	record2
3	record1
4	record3
```

**Code:**

```python
import hashlib

# Sample hash function for deduplication
def deduplicate(record):
    hash_key = hashlib.md5(record.encode()).hexdigest()
    if hash_key not in deduplication_map:
        deduplication_map[hash_key] = record
        return True  # New record, add to storage and send to processing pipeline
    else:
        return False  # Duplicate detected, no action needed

# Initialize a dictionary for storing hashes
deduplication_map = {}
```

**Example input / output:**

```text
Input Records:
[{"id": 1, "value": "record1"},
 {"id": 2, "value": "record2"},
 {"id": 3, "value": "record1"}, // Duplicate of first record
 {"id": 4, "value": "record3"}]

Deduplication Process:
Record 1: New -> Added to storage and sent for processing.
Record 2: New -> Added to storage and sent for processing.
Record 3: Duplicate of Record 1 -> Not added again, no action needed.
Record 4: New -> Added to storage and sent for processing.

Output Records after Deduplication:
[{"id": 1, "value": "record1"},
 {"id": 2, "value": "record2"},
 {"id": 4, "value": "record3"}]
```

**Data:**

```json
{
  "records": [
    {"id": 1, "value": "record1"},
    {"id": 2, "value": "record2"},
    {"id": 3, "value": "record1"}, // Duplicate of first record
    {"id": 4, "value": "record3"}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Record In
4. Is Deduplicated?
5. Send to Processing Pipeline
6. Store Record & Send toProcessing Pipeline
7. Processed
8. Stored in Deduplication Map

