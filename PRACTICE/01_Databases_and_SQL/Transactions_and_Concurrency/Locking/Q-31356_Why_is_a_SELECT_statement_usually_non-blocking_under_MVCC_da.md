---
type: "practice"
difficulty: "Deep"
---

# Why is a SELECT statement usually non-blocking under MVCC databases?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ MVCC](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/Locking/MVCC.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview
In MVCC (Multiversion Concurrency Control) databases, a SELECT statement typically does not block other transactions because it reads data from a version of the database that was committed at the time the transaction started. This is in contrast to INSERT, UPDATE, and DELETE statements which require locks on the data being modified.

#### Triage
The question asks why a SELECT statement usually remains non-blocking under MVCC databases, which means we need to understand how MVCC handles concurrent read operations without interfering with write transactions. We'll assume that MVCC databases use some form of transactional snapshot or versioning mechanism to achieve this. This is important for maintaining the consistency and performance of read operations in a multi-user environment.

#### First-Principles Solution

Understanding MVCC: In an MVCC database, each write operation (insert, update, delete) generates a new version of the data. Each transaction sees only the versions of data that were present at the time the transaction started.
Transaction Start and Isolation Level: When a SELECT statement starts, it captures the state of the database as of its start time. This snapshot is used to read all data that was committed before this point in time.
Non-Blocking Reads: Since a SELECT reads only from versions of the data that were present at the transaction's start time, and these versions are immutable (no other transactions can modify them), there's no risk of interference with ongoing write operations.
Handling Uncommitted Data: If a write operation occurs after the start of a read transaction but before it commits, MVCC ensures that this uncommitted data is not visible to the SELECT statement.

#### Closing Summary
and Common Mistakes: A SELECT statement in an MVCC database remains non-blocking because it operates on a snapshot of the database state at its start. This approach avoids blocking other transactions by ensuring reads do not conflict with writes. However, developers must be aware that uncommitted data is invisible to read transactions during their execution.

#### Common Mistakes/Follow-Ups

Ignoring Uncommitted Data: Assuming that uncommitted writes are visible to SELECT statements can lead to incorrect queries and inconsistent results.
Incorrect Isolation Levels: Misunderstanding the isolation level settings (like Serializable) can inadvertently enable blocking behavior for read transactions.

**Comparison:**

| Transaction | Type   | Data Version |
|-------------|--------|--------------|
| T1          | SELECT | 5            |
| T2          | INSERT | 6            |

**Code:**

```sql
-- Example of a SELECT statement in MVCC
SELECT * FROM users WHERE id = 1;
```

**Example input / output:**

```text
Input:
- Transaction T1 starts at time t0 and executes SELECT * FROM users WHERE id = 1.
- At t0, the database snapshot includes version 5 of user data.

Output:
- T1 reads version 5 of the data without blocking any transactions that occur after t0 but before committing.
```

**Data:**

```json
{
  "database": {
    "start_time": "2023-10-01T10:00:00Z",
    "transactions": [
      {"id": 1, "type": "SELECT", "data_version": 5},
      {"id": 2, "type": "INSERT", "data_version": 6}
    ]
  }
}
```

**Flow:**

1. Yes
2. No
3. Transaction T1 Starts
4. Captures Snapshot at t0
5. Data Version < 6?
6. Reads Version 5 Data
7. Blocks on Commit of NewData

