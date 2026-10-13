---
type: "practice"
difficulty: "Intermediate"
---

# How does SQL Server's deadlock monitor decide which session becomes the victim and how can you influence that with DEADLOCK_PRIORITY?

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ Deadlock Causes and Diagnosis](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/Deadlocks/Deadlock_Causes_and_Diagnosis.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In SQL Server, deadlocks occur when two or more sessions are waiting on each other to release locks, creating a circular wait condition. The deadlock monitor detects these conditions and resolves them by terminating one of the sessions. This is done based on the DEADLOCK_PRIORITY setting of the transactions involved. Understanding how the deadlock monitor decides which session becomes the victim can help you avoid or mitigate deadlocks through strategic transaction priority management.

#### Triaging the Question

SQL Server’s deadlock detection mechanism involves monitoring a transaction's execution state and lock requests against existing locks held by other sessions. The primary goal is to terminate one of the transactions to break the circular wait condition, but not always the "first offender" or "least important." SQL Server uses DEADLOCK_PRIORITY to influence which session gets terminated.

#### First-Principles Solution

Understanding DEADLOCK_PRIORITY:

DEADLOCK_PRIORITY can be set on a per-session basis using the SET DEADLOCK_PRIORITY command.
Valid values are:
0 (default): No special priority.
-1 to 5: Higher numbers indicate higher priority.

Deadlock Detection Process:

The deadlock monitor periodically checks for circular waits by analyzing transaction graphs.
If a cycle is detected, the monitor selects one of the transactions as the victim based on the DEADLOCK_PRIORITY setting and the transaction isolation level (Serializable or Snapshot).

Victim Selection Criteria:

Sessions with higher DEADLOCK_PRIORITY are less likely to be chosen as victims.
Transactions that have held locks longer (i.e., been running for a longer time) may be selected more frequently.

Impact of Transaction Isolation Level:

Serializable transactions tend to hold more locks, making them harder to break and thus increasing the chance they might become victims.
Snapshot isolation can sometimes reduce lock contention but still relies on DEADLOCK_PRIORITY.

#### Closing Summary

In summary, SQL Server's deadlock monitor uses a combination of transaction priority (DEADLOCK_PRIORITY) and execution time to decide which session becomes the victim. By setting appropriate priorities, you can influence which transactions are more likely to be terminated in case of deadlocks.

#### Common Mistakes/Follow-Ups
Over-relying on DEADLOCK_PRIORITY: While it helps, overly high-priority settings might lead to unintended transaction terminations.
Ignoring Transaction Isolation Levels: Serializable and Snapshot isolation levels can significantly affect lock behavior and deadlock resolution.
Failing to Monitor Deadlock Frequency: Regularly checking for deadlocks and understanding their patterns is crucial.

**Code:**

```sql
-- Setting DEADLOCK_PRIORITY on a session
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
BEGIN TRAN;

-- Example setting high priority
SET DEADLOCK_PRIORITY HIGH;

-- Other transactions can set different priorities to influence deadlock resolution
```

**Example input / output:**

```text
-- Sample input for a transaction with high priority
BEGIN TRAN;
SELECT * FROM Orders WHERE OrderID = 12345 FOR UPDATE; -- Locks the row
SET DEADLOCK_PRIORITY HIGH;

-- Another session might run:
BEGIN TRAN;
SELECT * FROM Orders WHERE OrderID = 67890 FOR UPDATE; -- Blocks on locked row
```

**Flow:**

1. Priority Selection
2. Yes
3. No
4. Transaction Graph
5. Deadlock Detected?
6. Select Victim
7. Terminate Transaction
8. Break Cycle
9. Continue Monitoring
10. High Priority Transactions LessLikely to be Victims
11. Transactions Held LongestMore Likely to be Victims

