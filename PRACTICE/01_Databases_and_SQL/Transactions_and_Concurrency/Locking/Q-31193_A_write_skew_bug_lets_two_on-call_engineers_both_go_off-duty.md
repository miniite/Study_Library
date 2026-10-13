---
type: "practice"
difficulty: "Expert"
---

# A write skew bug lets two on-call engineers both go off-duty. Show how you would reproduce it and which isolation change or explicit lock fixes it.

**Difficulty:** Expert · **Levels:** SDE-1, SDE-1/2 · [↑ Range Locks and Retries](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/Locking/Range_Locks_and_Retries.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a distributed database environment, write skew bugs can lead to unexpected concurrency issues, often manifesting as inconsistent state changes. In this scenario, two on-call engineers both attempt to resolve an issue and end up off-duty due to a write skew bug. To reproduce and fix this issue, we need to understand the root cause and apply appropriate isolation levels or explicit locking mechanisms.

#### Triage

The question is asking for a step-by-step approach to identify and mitigate a write skew bug in a database system. The core of the problem involves understanding concurrent transactions that lead to inconsistent states and how to reproduce this issue in a controlled environment. We need to consider the current isolation level, transaction boundaries, and potential lock scopes.

#### First-Principles Solution
1. Understand the Scenario

The engineers are resolving an issue (likely a bug or anomaly) in the database that involves modifying data. The state of the system is such that both engineers, upon seeing the same pending update, decide to proceed with their own modifications, leading to off-duty decisions.

2. Isolation Levels and Write Skew

In multi-version concurrency control (MVCC), write skew typically occurs when two or more transactions read a snapshot of the database at different times but see inconsistent states. This can lead to conflicts where both transactions attempt to modify the same data, resulting in an undesirable state.

3. Reproducing the Bug

To reproduce this bug, we need:

A multi-versioned database system with MVCC.
Two concurrent transactions (T1 and T2) that read different versions of the same data.
Both transactions modifying the same piece of data but based on different snapshots.
The final state leading to an unexpected outcome.
4. Identifying the Issue

Let's assume we have a table issues with columns: id, status, and resolution. Initially, both engineers see that issue 10 is in status 'open' with no resolution. Both decide to resolve it but based on different data versions:

Engineer A reads an older version where the resolution field is null.
Engineer B reads a newer version where another engineer has already started working on resolving the issue.
5. Fixing the Bug

To fix this, we can either adjust the isolation level or add explicit locks to ensure that such conflicts are avoided:

Adjust Isolation Level: Increase the isolation level from read committed (RC) to serializable (SN). This will serialize transactions and prevent conflicting operations.
Explicit Locking: Ensure that both engineers acquire a write lock on the issues table before modifying it. A two-phase locking protocol can be used where each engineer acquires an exclusive lock, ensuring no concurrent updates.

#### Closing Summary

By understanding the isolation levels and concurrency control mechanisms in place, we can identify and address write skew bugs. Adjusting isolation levels or implementing explicit locking can help prevent inconsistent states caused by concurrent modifications. The key is to ensure that all operations are serialized or properly synchronized to avoid conflicts.

#### Common Mistakes/Follow-Ups
Incorrect Isolation Level: Choosing the wrong isolation level (e.g., repeatable read instead of serializable) can lead to false positives.
Insufficient Locking: Failing to implement proper locking mechanisms can result in race conditions and inconsistent states.

**Comparison:**

| id | status | resolution |
|----|--------|------------|
| 10 | open   | null       |

**Code:**

```sql
-- Example SQL for setting isolation levels

-- Set transaction isolation level to serializable
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;

-- Explicitly acquire locks
BEGIN;
SELECT * FROM issues WHERE id = 10 FOR UPDATE; -- Acquire write lock
UPDATE issues SET resolution = 'resolution_1' WHERE id = 10;
COMMIT;

BEGIN;
SELECT * FROM issues WHERE id = 10 FOR UPDATE; -- Acquire write lock
UPDATE issues SET resolution = 'resolution_2' WHERE id = 10;
COMMIT;
```

**Example input / output:**

```text
- **Input**: Two transactions both read and update issue 10.
- **Output**: Both transactions successfully resolve the issue, but one is marked as resolved twice or the second transaction fails due to a lock conflict.
```

**Data:**

```json
{
  "issues": [
    {
      "id": 10,
      "status": "open",
      "resolution": null
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Transaction T1 Starts
6. Read Issue 10
7. Check if Resolution is Null
8. Update Issue with Resolution 1
9. Resolve Issue in Another Way
10. Commit Transaction
11. Finish
12. Transaction T2 Starts
13. Read Issue 10
14. Check if Resolution is Null
15. Update Issue with Resolution 2
16. Resolve Issue in Another Way
17. Commit Transaction

