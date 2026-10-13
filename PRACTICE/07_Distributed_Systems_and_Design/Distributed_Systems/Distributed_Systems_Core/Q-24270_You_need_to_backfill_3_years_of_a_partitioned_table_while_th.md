---
type: "practice"
difficulty: "Deep"
---

# You need to backfill 3 years of a partitioned table while the daily pipeline keeps running on today's partition. Explain how you isolate the backfill so it doesn't contend with the live job and keep both idempotent per partition.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Distributed Backfills](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Clock_Skew_and_Time_Windows/Distributed_Backfills.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of backfilling 3 years of a partitioned table while ensuring the daily pipeline keeps running smoothly, we need to design an isolation strategy that prevents contention between the two processes. This involves carefully managing transactional boundaries and ensuring idempotency per partition. Let's break down the steps to achieve this.

#### Triage
Understanding the Problem

The primary goal is to perform a backfill operation on historical data without disrupting ongoing, live data processing tasks. Both the backfill and daily pipeline should handle each partition independently, ensuring no conflicts arise between them. We need to ensure that each partition can be processed by both pipelines without causing issues such as duplicate records or data corruption.

#### Assumptions
The table is partitioned by date.
There are multiple partitions corresponding to different days.
Both the backfill and daily pipeline processes operate on a per-partition basis.
We have access to appropriate database management system (DBMS) features for transactional control, such as transaction isolation levels and locks.

#### First Principles Solution
Transaction Management: Use DBMS transaction management capabilities to ensure that both pipelines can run concurrently without interfering with each other.
Partition-Level Isolation: Ensure that the backfill operation only affects historical partitions and does not interfere with currently processing partitions.
Idempotency Per Partition: Design the pipeline logic such that it can re-run on a partition multiple times without altering the end result.
Step-by-Step Solution

Create a Separate Backfill Job:

Define a new job specifically for backfilling historical data. This job will focus solely on older partitions, ensuring that current processing is not impacted.

Transaction Boundaries:

Use transactional boundaries to encapsulate the operations within each partition. For example, in SQL, use BEGIN TRANSACTION and COMMIT statements to ensure atomicity.

Partition-Specific Logic:

Implement logic that checks if a partition has already been processed by either pipeline. This can be done using a timestamp or a flag in the database.

Concurrency Control:

Utilize DBMS features like read locking (SELECT ... FOR UPDATE in SQL) to prevent concurrent updates on overlapping partitions.

Idempotency Design:

Ensure that each partition's processing logic is idempotent, meaning it can be re-executed without changing the outcome.

Monitoring and Validation:

Set up monitoring to track the progress of both backfill and daily pipelines. Validate the integrity of data after completion.

#### Example
Implementation

Here’s an example using SQL to illustrate the concept:

SQL
-- Step 1: Identify partitions for backfilling
SELECT DISTINCT partition_date FROM historical_data WHERE partition_date < '2023-01-01';

-- Step 2: Process each partition in a transactional context
BEGIN TRANSACTION;

-- Backfill logic per partition
UPDATE historical_data SET processed_flag = true WHERE partition_date = '2022-12-01';
INSERT INTO historical_data SELECT * FROM old_source_table WHERE date >= '2022-12-01';

COMMIT;

#### Closing Summary
and Common Mistakes

To summarize, the key is to ensure that both backfilling and daily pipelines operate independently on different partitions. By using transactional boundaries and ensuring idempotency, we can avoid contention and data integrity issues.

#### Common mistakes
include:

Not isolating partitions properly, leading to conflicts.
Failing to make pipeline logic idempotent, causing unintended side effects during reprocessing.
Overlooking the need for monitoring and validation after backfilling is complete.

**Code:**

```sql
-- Example SQL script for backfilling
BEGIN TRANSACTION;
UPDATE historical_data SET processed_flag = true WHERE partition_date = '2022-12-01';
INSERT INTO historical_data SELECT * FROM old_source_table WHERE date >= '2022-12-01';
COMMIT;
```

**Data:**

```sql
-- Sample data schema
CREATE TABLE historical_data (
    id INT PRIMARY KEY,
    partition_date DATE NOT NULL,
    processed_flag BOOLEAN DEFAULT FALSE,
    -- other columns
);
```

**Flow:**

1. Backfill Job Starts
2. Identify Partitions
3. Create Transactional Context
4. Process Partition
5. Mark as Processed
6. Update Database
7. End Transaction
8. Daily Pipeline ContinuesUnaffected

