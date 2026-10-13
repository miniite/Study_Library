---
type: "practice"
difficulty: "Deep"
---

# Your lake ingestion is idempotent on paper but you keep seeing duplicate training rows after retries. Walk me through where the guarantee actually breaks.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ GDPR Deletion in Immutable Lakes](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/GDPR_Deletion_in_Immutable_Lakes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with an idempotent data ingestion system, it's crucial to understand how and why a supposedly foolproof mechanism might still fail in practice. In this scenario, we're seeing duplicates being ingested despite our design ensuring that the same operation can be retried without additional side effects. Let's break down where the guarantee of idempotence breaks.

#### Triage
What the Question is Really Asking

The question is probing into the implementation details and potential pitfalls of an idempotent data ingestion system in a data architecture, particularly focusing on how it interacts with systems like data lakes and warehouses. The key points to consider are:

Idempotence guarantee: Whether the same operation can be safely retried without changing the final state.
Duplicate rows issue: Why duplicate training rows are being ingested despite retries.

#### Assumptions
We have a system that is designed to be idempotent in theory, meaning that performing an operation multiple times should not change the outcome beyond the first execution.
The idempotence is maintained at the data source level and during the ingestion process.
There are no issues with downstream systems or processes.
Reasoning from First Principles
Ingestion Process
Data Source: Assume we have a source that generates training rows, such as logs or event streams.
Ingestion System: The system ingests these data points into a data lake or warehouse using an idempotent mechanism. This could involve checksums, timestamps, and other unique identifiers to ensure uniqueness.
Why Duplicates Occur
Network Issues:
Network interruptions can cause retries that fail due to timing issues. For instance, the first write might succeed but a subsequent retry might collide with another concurrent write.
Concurrency: In distributed systems, race conditions can lead to duplicate entries if multiple instances of the ingestion system attempt to process and commit data simultaneously without proper coordination.
Checksum Validation Failures:
If checksums are used for validation, they might fail due to transient errors in computation or network issues, leading to retries that succeed.
Timestamp Conflicts: If timestamps are relied upon as unique identifiers, clock skew between different nodes can cause duplicate entries when they interpret the same timestamp.

#### Example Scenario

#### Cons
ider a scenario where an ETL job ingests data from a stream into a data lake:

Source generates row A at time t1.
Ingestion process starts, and row A is written to the data lake with checksum C.
Network interruption causes ingestion to retry. Row A is again attempted for insertion.
Concurrent writes by another node also attempt to insert row A, leading to a conflict.

#### Closing Summary
+ Common Mistakes/Follow-Ups

The idempotence guarantee can break due to various factors like network issues, concurrency, checksum validation failures, and timestamp conflicts. To mitigate these issues, consider the following:

Synchronized Writes: Use distributed locks or semaphores to ensure that only one instance writes at a time.
Retry Mechanisms with Backoffs: Implement exponential backoff strategies to avoid overwhelming the system during retries.
Conflict Resolution Strategies: Design algorithms for detecting and resolving conflicts, such as using optimistic concurrency control.

#### Common Mistakes
Over-reliance on Checksums: Assuming that checksum validation alone guarantees idempotence can be risky due to transient errors.
Ignoring Concurrency Issues: Failing to handle race conditions properly can lead to duplicate entries even with idempotent designs.
Network and Infrastructure Assumptions: Assuming stable network infrastructure without considering edge cases like latency and packet loss.

**Comparison:**

```
Attempt	Action	Outcome
1	Write to database	Success
2	Write to database	Network error
3	Write to database	Success
```

**Code:**

```python
# Example of a simple retry mechanism with exponential backoff
import time

def safe_write(data, max_retries=5):
    for attempt in range(max_retries + 1):
        try:
            # Simulate write operation (e.g., insert into database)
            print(f"Writing data: {data}")
            break
        except Exception as e:
            if attempt < max_retries:
                backoff_time = 2 ** attempt
                print(f"Write failed, retrying in {backoff_time} seconds...")
                time.sleep(backoff_time)
            else:
                raise

safe_write("Row A")
```

**Example input / output:**

```text
Input:
- Data to be ingested: Row A

Output (Simulated retries):
Writing data: Row A
# Simulates a network issue, retrying...
Write failed, retrying in 2 seconds...
Writing data: Row A
Writing data: Row A
```

**Data:**

```json
{
  "rowA": {
    "data": "Example row data",
    "checksum": "C1234567890"
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Start Ingestion
6. Is Network Good?
7. Write Data
8. Check for Conflicts
9. Commit Write
10. Retry with Backoff
11. Retry with Backoff

