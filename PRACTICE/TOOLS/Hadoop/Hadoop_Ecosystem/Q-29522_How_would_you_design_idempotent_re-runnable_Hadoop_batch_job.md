---
type: "practice"
difficulty: "Deep"
---

# How would you design idempotent, re-runnable Hadoop batch jobs so that a failed midnight run can be safely retried without double-counting?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ YARN Troubleshooting](<../../../../TOOLS/Hadoop/Hadoop_Ecosystem/YARN_Troubleshooting.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design idempotent and re-runnable Hadoop batch jobs, we need to ensure that each job can handle being retried multiple times without causing any data inconsistencies or double-counting issues. The key idea here is to make the jobs aware of their previous state and to incorporate a mechanism for skipping duplicate work.

#### Triaging the Question

The core challenge in this question is ensuring idempotency, which means that rerunning the job should have no additional effect beyond its first execution. This involves tracking the progress and state of each batch processing step so that duplicates can be skipped if they are encountered again. The interviewer is probing your understanding of Hadoop’s MapReduce framework, distributed processing, and strategies to manage state across jobs.

#### Assumptions
Input Data: The input data is immutable and does not change between job executions.
Job Execution Context: The job can maintain some state information, possibly stored in a database or a metadata store like HDFS or a NoSQL database.
Error Handling: The system should be able to handle transient failures gracefully.

#### Solution

To achieve idempotency, we need to implement the following steps:

State Tracking: Maintain a list of processed records or keys in a durable storage mechanism.
Job Execution Logic:
Check if the current record/key has been processed before.
If not, process it and mark as completed.
Retrying Mechanism:
For failed jobs, re-run only those steps that have not been marked as complete.
Step-by-Step Solution

State Tracking: We can use HDFS, a NoSQL database (like Cassandra or DynamoDB), or even a simple text file stored in Hadoop Distributed File System (HDFS) to track processed records. For simplicity, let's assume we store these identifiers in an HDFS directory.

Job Execution Logic:

In the MapReduce job, each map task receives input data.
The mapper checks if the key has been processed before by looking up its state in the durable storage mechanism (e.g., reading from a text file or querying a database).
If the record is not marked as processed, it processes it and updates the state to mark it as completed. If it’s already marked as processed, the task simply skips processing.

Retrying Mechanism:

When a job fails, we can re-run the entire job or only specific tasks that have failed by checking their status against the state storage.

#### Example
Implementation
PYTHON
# Pseudocode for MapReduce Job

def map(key, value):
    # Check if key has been processed before
    if check_key_processed(key):
        return  # Skip processing if already done

    # Process the record
    process_record(key, value)

    # Update state to mark as completed
    update_state_storage(key)

# Pseudocode for State Tracking and Check

def check_key_processed(key):
    try:
        with open(f"/path/to/processed_records/{key}", "r") as file:
            return True  # Key has been processed before
    except FileNotFoundError:
        return False  # Key has not been processed before

def update_state_storage(key):
    with open(f"/path/to/processed_records/{key}", "w") as file:
        pass  # Mark key as processed (write any token to the file)

#### Closing Summary
and Common Mistakes

To ensure idempotency in Hadoop batch jobs, we need a reliable state-tracking mechanism that can be checked before processing each record. Using a combination of durable storage and careful logic ensures that jobs can be retried without causing double-counting or inconsistent data states.

#### Common mistakes
to avoid include:

Not maintaining a clear and consistent state representation.
Failing to handle failures gracefully, leading to data corruption.
Overcomplicating the solution with unnecessary overhead.

**Code:**

```python
# Example Python Pseudocode for MapReduce Job

def map(key, value):
    # Check if key has been processed before
    if check_key_processed(key):
        return  # Skip processing if already done

    # Process the record
    process_record(key, value)

    # Update state to mark as completed
    update_state_storage(key)

def check_key_processed(key):
    try:
        with open(f"/path/to/processed_records/{key}", "r"):
            return True  # Key has been processed before
    except FileNotFoundError:
        return False  # Key has not been processed before

def update_state_storage(key):
    with open(f"/path/to/processed_records/{key}", "w") as file:
        pass  # Mark key as processed (write any token to the file)
```

**Data:**

```json
{
  "processed_records": [
    "record1",
    "record2"
  ]
}
```

