---
type: "practice"
difficulty: "Expert"
---

# What are the UPDLOCK HOLDLOCK and READPAST table hints and give a real scenario for each in a queue-processing pattern.

**Difficulty:** Expert · **Levels:** SDE-1, SDE-1/2 · [↑ MERGE Pitfalls and Error Handling](<../../../../TOOLS/SQL_Server/T-SQL/MERGE_Pitfalls_and_Error_Handling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

UPDLOCK, HOLDLOCK, and READPAST are T-SQL table hints used to control locking behavior during SQL Server operations. UPDLOCK ensures that a row is locked until the transaction completes, while HOLDLOCK maintains the lock on the entire result set. On the other hand, READPAST allows you to skip over rows being updated by another transaction. In queue-processing patterns, these hints can help prevent deadlocks and ensure data integrity.

Question Triage

The question is asking for a detailed explanation of three specific SQL Server table hints: UPDLOCK, HOLDLOCK, and READPAST. The interviewer wants to understand how you would use them in a queue-processing scenario. This involves explaining the behavior, benefits, and potential drawbacks of each hint.

#### Assumptions
We are working with a multi-user environment where multiple transactions might be accessing and modifying the same data.
The database schema includes a queue table that tracks tasks to be processed.
UPDLOCK Table Hint
Behavior: When used in an UPDATE statement, UPDLOCK locks the updated rows until the transaction is completed. This prevents other transactions from updating or deleting these locked rows until the current transaction completes.
Benefit: Ensures data integrity by preventing "losing updates" where multiple transactions might update the same row simultaneously.
Drawback: Can lead to deadlocks if not used carefully, as it locks up resources.
HOLDLOCK Table Hint
Behavior: Maintains an isolation level on all rows in the result set for the duration of the transaction. It is similar to using SET TRANSACTION ISOLATION LEVEL SERIALIZABLE but applies only within the context of a specific query.
Benefit: Ensures that once a row or rows are selected, they cannot be changed by other transactions until the current transaction completes.
Drawback: Can significantly impact performance due to increased locking and resource usage.
READPAST Table Hint
Behavior: Skips over rows being updated by another transaction. It does not lock the skipped rows, allowing your query to proceed as if they were not present.
Benefit: Prevents blocking where a transaction would otherwise have to wait for an update operation on a row it is interested in to complete.
Drawback: Can lead to incomplete reads and potential data loss if you are expecting certain updates.
Scenario: Queue Processing Pattern

#### Cons
ider a scenario where we have a queue table with tasks that need to be processed. Each task has a status field indicating its current state (e.g., pending, processing, or completed).

#### Example
Schema
SQL
CREATE TABLE QueueTable (
    Id INT PRIMARY KEY,
    TaskName NVARCHAR(100),
    Status NVARCHAR(50)
);
Using UPDLOCK in a Queue Processing Pattern

To ensure that only one transaction processes a specific task at a time, you can use the UPDLOCK hint:

SQL
BEGIN TRANSACTION;

-- Select and update with UPDLOCK to lock the row until the end of the transaction.
SELECT TOP 1 * FROM QueueTable WHERE Status = 'pending' AND LockStatus = 0 WITH (UPDLOCK) FOR UPDATE;
UPDATE QueueTable SET Status = 'processing', LockStatus = 1 WHERE Id = @Id;

-- Process the task...

COMMIT TRANSACTION;
Using HOLDLOCK in a Queue Processing Pattern

To ensure that all selected tasks remain locked until the transaction completes, even if multiple rows are processed:

SQL
BEGIN TRANSACTION;

-- Select and update with HOLDLOCK to maintain an isolation level on the entire result set.
SELECT * FROM QueueTable WHERE Status = 'pending' WITH (HOLDLOCK) FOR UPDATE;
UPDATE QueueTable SET Status = 'processing', LockStatus = 1 WHERE Id IN (@Ids);

-- Process tasks...

COMMIT TRANSACTION;
Using READPAST in a Queue Processing Pattern

To avoid blocking and ensure that you do not miss out on processing any pending tasks:

SQL
BEGIN TRANSACTION;

-- Select with READPAST to skip over rows being updated by another transaction.
SELECT * FROM QueueTable WHERE Status = 'pending' AND LockStatus = 0 WITH (READPAST);
UPDATE QueueTable SET Status = 'processing', LockStatus = 1 WHERE Id IN (@Ids);

-- Process tasks...

COMMIT TRANSACTION;

#### Closing Summary
and Common Mistakes/Follow-Ups

In queue-processing patterns, using UPDLOCK, HOLDLOCK, and READPAST can help manage concurrency issues effectively. However, improper use can lead to deadlocks or incomplete processing.

#### Common mistakes
include:

Not understanding the implications of locking on performance.
Failing to properly manage transaction boundaries.
Overusing HOLDLOCK, which can degrade system performance due to excessive locking.

For follow-ups, you might be asked about optimizing the queue mechanism, handling failures, or integrating these hints with other concurrency control strategies like optimistic locking or distributed transactions.

**Comparison:**

| Id | TaskName   | Status    | LockStatus |
|----|-----------|----------|------------|
| 1  | TaskA     | processing | 1          |
| 2  | TaskB     | pending   | 0          |
| 3  | TaskC     | processing | 1          |
| 4  | TaskD     | completed | 0          |

**Code:**

```sql
-- Example using UPDLOCK
BEGIN TRANSACTION;

SELECT TOP 1 * FROM QueueTable WHERE Status = 'pending' AND LockStatus = 0 WITH (UPDLOCK) FOR UPDATE;
UPDATE QueueTable SET Status = 'processing', LockStatus = 1 WHERE Id = @Id;

COMMIT TRANSACTION;

-- Example using HOLDLOCK
BEGIN TRANSACTION;

SELECT * FROM QueueTable WHERE Status = 'pending' WITH (HOLDLOCK) FOR UPDATE;
UPDATE QueueTable SET Status = 'processing', LockStatus = 1 WHERE Id IN (@Ids);

COMMIT TRANSACTION;

-- Example using READPAST
BEGIN TRANSACTION;

SELECT * FROM QueueTable WHERE Status = 'pending' AND LockStatus = 0 WITH (READPAST);
UPDATE QueueTable SET Status = 'processing', LockStatus = 1 WHERE Id IN (@Ids);

COMMIT TRANSACTION;
```

**Example input / output:**

```text
-- Using UPDLOCK
BEGIN TRANSACTION;

SELECT TOP 1 * FROM QueueTable WHERE Status = 'pending' AND LockStatus = 0 WITH (UPDLOCK) FOR UPDATE;
UPDATE QueueTable SET Status = 'processing', LockStatus = 1 WHERE Id = 1;

COMMIT TRANSACTION;

-- Using HOLDLOCK
BEGIN TRANSACTION;

SELECT * FROM QueueTable WHERE Status = 'pending' WITH (HOLDLOCK) FOR UPDATE;
UPDATE QueueTable SET Status = 'processing', LockStatus = 1 WHERE Id IN (2, 3);

COMMIT TRANSACTION;

-- Using READPAST
BEGIN TRANSACTION;

SELECT * FROM QueueTable WHERE Status = 'pending' AND LockStatus = 0 WITH (READPAST);
UPDATE QueueTable SET Status = 'processing', LockStatus = 1 WHERE Id IN (4, 5);

COMMIT TRANSACTION;
```

**Data:**

```sql
CREATE TABLE QueueTable (
    Id INT PRIMARY KEY,
    TaskName NVARCHAR(100),
    Status NVARCHAR(50),
    LockStatus BIT
);
INSERT INTO QueueTable (Id, TaskName, Status, LockStatus) VALUES
(1, 'TaskA', 'pending', 0),
(2, 'TaskB', 'pending', 0),
(3, 'TaskC', 'processing', 1),
(4, 'TaskD', 'completed', 0);
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Queue Table
6. Task Status = 'pending'?
7. (SELECT with UPDLOCK)
8. UPDATE
9. Mark as Processing
10. Task Status = 'processing' &LockStatus = 1?
11. (SELECT with HOLDLOCK)
12. UPDATE
13. Mark as Processing
14. Task Status = 'pending' &LockStatus = 0?
15. (SELECT with READPAST)
16. UPDATE
17. Mark as Processing

