---
type: "practice"
difficulty: "Deep"
---

# How do you design CDC ingestion so a downstream backfill and the live stream don't clobber each other's rows during the overlap window?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ CDC Fundamentals and Ordering](<../../../../02_Data_Engineering/Pipelines_and_Integration/Change_Data_Capture/CDC_Fundamentals_and_Ordering.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design Change Data Capture (CDC) ingestion where both a live stream and a backfill operation do not interfere with each other, we need to carefully manage the data streams and ensure they are processed in a way that avoids duplicates or lost rows during their overlap window. This involves designing a robust system architecture and implementing appropriate data handling mechanisms.

#### Triage

The core of this question is about ensuring data integrity and consistency when merging live CDC events with historical backfill data. The interviewer is probing your understanding of concurrency control, data deduplication, and the overall design principles to prevent conflicts during overlapping operations.

Step-by-Step Solution

Identify Key Components:

Live Stream: Continuous stream of data changes from a source system.
Backfill Data: Historical records that need to be ingested into the database before or during live streaming starts.

Define the Overlap Window:

The overlap window is the period when both the backfill and the live stream are active, potentially capturing overlapping transactions.

Deduplication Strategy:

Use a unique identifier (e.g., a primary key) to track records.
Maintain a local state or in-memory cache to store incoming rows temporarily until they can be safely processed.

Concurrency Control Mechanism:

Implement a mechanism to lock records during processing to prevent multiple processes from modifying the same record simultaneously.
Use timestamps or sequence numbers to detect and handle concurrent updates gracefully.

Data Processing Flow:

Filter out duplicate rows based on unique identifiers before storing them in the database.
Ensure that each row is processed only once by maintaining a set of already seen keys.

Error Handling and Logging:

Log any failures or conflicts to track down issues quickly.
Implement retry mechanisms for transient errors.
Detailed Steps

Set Up a Unique Identifier:

SQL
CREATE TABLE source_table (
    id INT PRIMARY KEY,
    column1 VARCHAR(255),
    column2 INT,
    -- other columns
    cdc_timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

Ingest Backfill Data:

Use a batch job to ingest historical data into the target database.
Ensure that backfill records are processed first before starting live streaming.

Establish a Processing Pipeline:

PYTHON
import json

def process_row(row):
    # Check if the row already exists in the cache or database
    if row_id not in seen_rows:
        # Insert/Update the row in the target database
        insert_into_target_database(row)
        seen_rows.add(row_id)

for row in live_stream:
    process_row(json.loads(row))

Concurrency Control:

Use a locking mechanism (e.g., SELECT ... FOR UPDATE) to lock rows during processing.
Ensure that the same row is not processed by multiple instances simultaneously.

#### Closing Summary

By carefully designing the ingestion pipeline, implementing robust concurrency controls, and ensuring proper deduplication strategies, we can prevent data clobbering between live streaming and backfill operations. This ensures that both historical and real-time data are accurately captured without duplication or loss during the overlap window.

#### Common Mistakes/Follow-Ups
Failing to account for concurrent updates: Ensure you have a mechanism in place to handle conflicts.
Not managing state properly: Use a cache or database to keep track of processed records.
Ignoring error handling: Implement retry logic and logging for better fault tolerance.

**Comparison:**

```
Unique Identifier	Row Data
1	{"id": 1, "value": "A"}
2	{"id": 2, "value": "B"}
3	{"id": 3, "value": "C"}
```

**Code:**

```python
def process_row(row):
    row_id = get_unique_identifier(row)
    if row_id not in seen_rows:
        insert_into_target_database(row)
        seen_rows.add(row_id)

for row in live_stream:
    process_row(json.loads(row))
```

**Flow:**

1. No
2. Yes
3. Start
4. Is Row Processed?
5. Insert Row into Target DB
6. Add to Seen Rows Set
7. End

