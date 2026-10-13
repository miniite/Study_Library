---
type: "practice"
difficulty: "Deep"
---

# How would you make your data pipeline deploy idempotent so that re-running the CD job (after a transient failure) doesn't double-insert rows or re-trigger a backfill?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Idempotent Deploy Jobs](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Idempotent_Deploy_Jobs/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we want to ensure that our CI/CD pipeline is deploy idempotent, meaning it can be run multiple times without causing unintended side effects like double-inserting rows or re-triggering a backfill after a transient failure. To achieve this, we need to introduce mechanisms to detect and handle already processed data effectively.

First, let's break down the problem:

Transient failures are temporary issues that could cause jobs to fail but should not impact the final state of the system.
Idempotency means running a job multiple times will have the same effect as running it once.
Double-inserts and re-triggered backfills can lead to data inconsistencies or inefficiencies.

To solve this, we need to implement two main strategies: idempotent operations in the code and tracking processed data externally.

Triaging the Problem

The core of the question is about making sure that a CI/CD pipeline job is idempotent when re-run after a transient failure. This means ensuring that any state changes (like inserting rows) are only applied once, even if the job runs multiple times.

We will assume:

The data pipeline involves ETL operations.
Transient failures can occur during data processing but do not affect the final state of the system.
The goal is to avoid double-inserts and unnecessary re-processing of data.

#### First-Principles Solution
Identify Unique Keys: Determine unique keys or identifiers for each row in your dataset that will be used to detect duplicates.
External Tracking System: Use a database or external storage (like a Redis set) to track processed rows or batches.
Idempotent Logic in Code: Modify the ETL job to check if a row has already been processed before inserting it.
Step-by-Step Solution

Identify Unique Keys:

For example, if you are ingesting data from a CSV file with columns id, name, and timestamp, id could be the unique key.
Ensure that the unique keys are stable and consistent across runs.

External Tracking System:

Use an external storage system like Redis to store processed keys. This can be done using a set or list data structure.
Each time the ETL job processes a row, it checks if the key is already in the tracking system before proceeding with any insertions.

Idempotent Logic in Code:

Modify your ETL script to include a check for processed keys.
PYTHON
import redis

r = redis.Redis(host='localhost', port=6379, db=0)

def process_row(row):
    key = row['id']

    if not r.sismember('processed_keys', key):  # Check if the key is in the set
        r.sadd('processed_keys', key)  # Add the key to the set (only once)
        insert_into_database(row)  # Insert the row into the database

def insert_into_database(row):
    # Your ETL logic here
    pass

Handling Transient Failures:

Ensure that your tracking system can handle transient failures gracefully.
Use idempotent operations for updates and inserts to avoid conflicts.

#### Closing Summary

By implementing these strategies, we ensure that our data pipeline is robust against transient failures while maintaining idempotency. This approach minimizes the risk of double-inserts or unnecessary re-processing, ensuring the integrity and consistency of the final dataset.

#### Common Mistakes/Follow-Ups
Failing to Use Idempotent Operations: Ensuring all database operations are idempotent is crucial.
Incorrect Tracking Mechanism: Using a tracking mechanism that does not handle transient failures properly can lead to false negatives or positives.
Over-engineering: Overcomplicating the solution with additional layers of logic when simpler solutions suffice.

**Comparison:**

```
Step	Action
1	Identify unique keys.
2	Use an external tracking system (e.g., Redis).
3	Implement idempotent logic in the ETL script.
```

**Code:**

```python
import redis

r = redis.Redis(host='localhost', port=6379, db=0)

def process_row(row):
    key = row['id']

    if not r.sismember('processed_keys', key):  # Check if the key is in the set
        r.sadd('processed_keys', key)  # Add the key to the set (only once)
        insert_into_database(row)  # Insert the row into the database

def insert_into_database(row):
    # Your ETL logic here
    pass
```

**Example input / output:**

| Input | Output                |
|-------|-----------------------|
| Row 1 (ID: 1) | Processed       |
| Row 2 (ID: 2) | Inserted into database, marked as processed |
| Re-run with same data | No new insertions or updates |

**Data:**

```json
{
  "data": [
    {"id": 1, "name": "Alice", "timestamp": 1635789024},
    {"id": 2, "name": "Bob", "timestamp": 1635789025}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Is Key Processed?
5. Moved to next row
6. Insert Row into Database
7. Marks Key as Processed inRedis
8. End

