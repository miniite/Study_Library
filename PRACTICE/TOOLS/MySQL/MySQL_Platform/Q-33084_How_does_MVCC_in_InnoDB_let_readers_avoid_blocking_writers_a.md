---
type: "practice"
difficulty: "Expert"
---

# How does MVCC in InnoDB let readers avoid blocking writers, and what are undo logs?

**Difficulty:** Expert · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Read Your Writes in MySQL](<../../../../TOOLS/MySQL/MySQL_Platform/Read_Your_Writes_in_MySQL.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

MVCC (Multi-Version Concurrency Control) is a key feature of InnoDB that allows multiple versions of data to exist simultaneously, enabling readers to see consistent snapshots without locking writers. This mechanism is crucial for achieving high concurrency in database operations.

#### Triage
and Assumptions

The question asks about how MVCC works in InnoDB and specifically what undo logs are used for. The key points to cover include:

How MVCC ensures non-blocking reads.
What an undo log is and its role in transaction management.

#### Example scenario
s illustrating the interaction between readers, writers, and the undo logs.

#### First-Principles Solution
Core Mechanism of MVCC

In InnoDB, each row version has a timestamp that tracks when it was inserted or updated. When a writer starts a transaction, it sets its own transaction ID. Readers can see different versions of data based on their snapshot timestamp, which is the maximum transaction ID seen by the reader.

How Readers Avoid Blocking Writers
Snapshot Reads: During a read operation, a transaction takes a snapshot of the database state at the time it began. This allows readers to see consistent data without needing locks.
Versioning Rows: When writing to a row, InnoDB writes the new version with a higher timestamp and keeps the old versions (which are marked as "visible" or "unexpired"). These older versions can be read by transactions that started before the write operation.
Role of Undo Logs

Undo logs store the old values of rows so that they remain accessible for reading even after being overwritten. Here’s how they work:

Transaction Starts: When a transaction begins, it starts with a snapshot of the database.
Row Updates: If a row is updated within a transaction, InnoDB writes an undo log entry containing the old value and a pointer to where the row is stored in the tablespace.
Conflict Detection: Readers can see older versions by looking at the undo logs if necessary.
Detailed Example

#### Cons
ider a scenario with two transactions: T1 writing to a row, and T2 reading from it:

At time ( t ), T1 starts a transaction and reads the initial value of the row.
At ( t+1 ), T1 updates the row. An undo log entry is created with the old value (let's say 10) at timestamp ( t+1 ).
At ( t+2 ), T2 starts a transaction and reads the same row. It sees the latest snapshot, which includes the new value (let's say 20). However, since it started after the write operation, it can still see the old value in the undo log if needed.

#### Closing Summary

MVCC ensures that readers do not block writers by allowing them to read older versions of data from the undo logs. The use of undo logs enables transactions to maintain consistency and concurrency without locking rows during reads. Misunderstanding these mechanisms often leads to issues like stale reads or transaction aborts due to row-level locks.

#### Common Mistakes
Overlooking Undo Logs: Not recognizing that old values are stored in the undo log.
Misinterpreting Snapshot Reads: Thinking readers always see the latest version, missing how they can access older versions via transactions.

**Comparison:**

| Timestamp | Row ID | Value |
|-----------|--------|-------|
| 2023-10-01T14:05:00Z | 1 | 10 |
| 2023-10-01T14:10:00Z | 1 | 20 |
| 2023-10-01T14:15:00Z | 1 | 30 |

**Code:**

```sql
-- Example of setting a transaction with MVCC
START TRANSACTION;

SELECT * FROM my_table WHERE id = 1 FOR UPDATE; -- Locks the row for update

UPDATE my_table SET value = 20 WHERE id = 1; -- Writes new value and stores old in undo log

COMMIT;
```

**Example input / output:**

```text
Transaction T1:
- START TRANSACTION
- SELECT * FROM my_table WHERE id = 1 FOR UPDATE
- UPDATE my_table SET value = 20 WHERE id = 1

Transaction T2 (reads after T1 commit):
- SELECT * FROM my_table WHERE id = 1 -- Reads the new value: 20

Transaction T3 (reads before T1 commits):
- SELECT * FROM my_table WHERE id = 1 FOR UPDATE
- UPDATE my_table SET value = 30 WHERE id = 1 -- New undo log entry for old value: 10, timestamp: 2023-10-01T14:15:00Z

Transaction T2 (reads after T3 commit):
- SELECT * FROM my_table WHERE id = 1 -- Reads the new value: 30
```

**Data:**

```json
{
  "transaction": {
    "id": 123,
    "start_timestamp": "2023-10-01T14:00:00Z",
    "operations": [
      { "type": "SELECT", "row_id": 1, "timestamp": "2023-10-01T14:05:00Z" },
      { "type": "UPDATE", "row_id": 1, "old_value": 10, "new_value": 20, "timestamp": "2023-10-01T14:10:00Z" }
    ]
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. No
6. Yes
7. Yes
8. No
9. Transaction Starts
10. Read Operation?
11. Is Snapshot Ready?
12. Wait for Locks
13. Update Row?
14. Write New Value, Store Old inUndo Log
15. Commit Transaction
16. Continue Reading
17. Write Operation?
18. Write New Value, Store Old inUndo Log
19. Transaction Commits?
20. Delete Undo Log Entry
21. Abort Transaction

