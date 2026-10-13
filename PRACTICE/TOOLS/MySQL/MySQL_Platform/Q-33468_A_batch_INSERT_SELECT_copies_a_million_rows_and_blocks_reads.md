---
type: "practice"
difficulty: "Intermediate"
---

# A batch INSERT ... SELECT copies a million rows and blocks reads on the source table — how do you reduce its lock footprint and isolation impact?

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ MySQL Performance Diagnosis](<../../../../TOOLS/MySQL/MySQL_Platform/MySQL_Performance_Diagnosis.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of high locking and reduced read performance caused by a large INSERT ... SELECT operation, we need to explore ways to minimize the lock footprint and ensure that reads from the source table are not blocked. The core idea is to decouple the write operation from its impact on reads.

The question is asking how to reduce the isolation impact of a batch insert operation in MySQL without compromising data integrity. We will assume the use of InnoDB as it supports row-level locking, which can mitigate some of the blocking issues but might still cause contention during large operations.

Let's start by understanding the assumptions:

The table has a significant number of rows (in this case, a million).
The INSERT ... SELECT operation is causing a high lock footprint and blocking reads.
We aim to find a balance between performance and data consistency.
Step-by-Step Solution

Split the Operation: Break down the large insert operation into smaller batches. This reduces the duration of the lock and minimizes its impact on concurrent reads.

Use Row-Level Locking: Ensure that row-level locking is being used by setting appropriate transaction isolation levels.

Optimize Indexes: Ensure indexes are optimized for the query to avoid full table scans, which can increase lock contention.

Parallel Inserts: Use parallel inserts or multi-threaded operations if possible to distribute the load and reduce the duration of exclusive locks.

Use Low-Isolation Levels: Consider using a lower isolation level (e.g., READ COMMITTED) for read-only transactions to allow concurrent reads while writing, but be cautious about data consistency.

Transaction Management: Use explicit transaction management to control when locks are acquired and released.

#### Example
Implementation

Here is an example implementation in SQL:

SQL
-- Step 1: Split the operation into smaller batches
WITH batched_data AS (
    SELECT * FROM source_table LIMIT 10000 OFFSET (page_number - 1) * 10000
)
INSERT INTO target_table (column1, column2, ...)
SELECT column1, column2, ... FROM batched_data;

-- Step 2: Use appropriate transaction isolation levels
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
START TRANSACTION;

-- Step 3: Perform the insert operation in a loop or using parallelism
REPEAT
    INSERT INTO target_table (column1, column2, ...)
    SELECT column1, column2, ... FROM batched_data LIMIT 10000 OFFSET (page_number - 1) * 10000;
UNTIL complete OR last_batch END REPEAT;

COMMIT;

#### Closing Summary

By splitting the operation into smaller batches and using appropriate transaction isolation levels, we can significantly reduce the lock footprint and minimize the impact on concurrent reads. However, it's crucial to monitor performance and adjust the batch size based on the specific workload and table structure.

#### Common Mistakes
and Follow-Ups
Not Splitting the Operation: Failing to split large operations into smaller batches can lead to high lock contention.
Using Inappropriate Isolation Levels: Using a higher isolation level than necessary can increase locking time, which might not be beneficial if reads are still needed during inserts.
Lack of Monitoring and Tuning: Without proper monitoring and tuning, the chosen solution might not perform optimally.

If there is any follow-up or additional context, such as specific table schema details or performance metrics, further adjustments can be made to fine-tune the approach.

