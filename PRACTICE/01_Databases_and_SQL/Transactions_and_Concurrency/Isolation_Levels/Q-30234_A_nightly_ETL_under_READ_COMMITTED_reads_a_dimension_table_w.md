---
type: "practice"
difficulty: "Intermediate"
---

# A nightly ETL under READ COMMITTED reads a dimension table while it's being updated, producing a mix of old and new rows. How do you make the read set consistent?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Phantoms and Double Booking](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Phantoms_and_Double_Booking.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue where an ETL process reading from a dimension table under READ COMMITTED isolation level encounters a mix of old and new rows during updates, we need to ensure that the read set remains consistent. This can be achieved by either upgrading the transaction isolation level or implementing a more robust data access strategy.

#### Triage

The question is essentially asking how to achieve a consistent snapshot of the dimension table while it's being updated. READ COMMITTED isolation ensures that transactions cannot read uncommitted data but can still see changes made by other transactions. However, if an ETL process starts reading from the middle of such an update, it may encounter a mix of old and new rows.

#### First-Principles Solution

To ensure consistency, we need to understand when READ COMMITTED is not sufficient and consider higher isolation levels or alternative strategies like locking mechanisms. For a Database Developer with intermediate experience, let's explore the options:

Upgrade Isolation Level: Increase the transaction isolation level from READ COMMITTED to REPEATABLE READ (RR) or SERIALIZABLE.
Locking Mechanisms: Use table-level or row-level locks to ensure that the ETL process reads a consistent snapshot.
Snapshot Isolation: Leverage database features like snapshot isolation if supported.
Detailed Solution
1. Upgrade to REPEATABLE READ (RR)

REPEATABLE READ ensures that once a transaction has read rows, those rows will not change until the transaction ends. This means the ETL process can read a consistent snapshot of the dimension table at the start and then use this snapshot throughout its execution.

SQL
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
-- Perform ETL operations here.
2. Locking Mechanisms

Alternatively, you can explicitly lock the rows before reading them to ensure they remain in a consistent state:

SQL
BEGIN TRANSACTION;

-- Acquire shared locks on the dimension table
SELECT * FROM dimension_table WITH (ROWLOCK, UPDLOCK);

-- Perform ETL operations here.

COMMIT;
3. Snapshot Isolation

If your database supports snapshot isolation, you can use it to read a consistent version of the data:

SQL
BEGIN TRANSACTION;

-- Start a transaction that uses SNAPSHOT isolation level
SET TRANSACTION ISOLATION LEVEL SNAPSHOT;

-- Perform ETL operations here.

COMMIT;

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, upgrading the isolation level to REPEATABLE READ or using explicit locking is effective for ensuring a consistent read set during an ETL process. However, be aware that higher isolation levels can increase contention and might slow down the database.

#### Common mistakes
include:

Assuming READ COMMITTED automatically provides a consistent snapshot.
Failing to understand the trade-offs between different isolation levels.
Not considering alternative strategies like locking mechanisms or snapshot isolation if available.

#### Follow-up
questions could involve discussing performance implications, choosing the right strategy based on workload characteristics, and ensuring that the chosen approach aligns with other database constraints such as concurrency requirements.

**Comparison:**

| id | name  |
|----|-------|
| 1  | Row A |
| 2  | Row B |
| 3  | Row C |

**Code:**

```

-- Example of setting transaction isolation level to REPEATABLE READ SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;

CODE
Copy

```sql
-- Example of using explicit locking
BEGIN TRANSACTION;
SELECT * FROM dimension_table WITH (ROWLOCK, UPDLOCK);
COMMIT;

-- Example of using snapshot isolation
BEGIN TRANSACTION;
SET TRANSACTION ISOLATION LEVEL SNAPSHOT;
SELECT * FROM dimension_table;
COMMIT;
```

**Example input / output:**

```text
-- Input: Setting isolation level to REPEATABLE READ and performing ETL operations
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
SELECT * FROM dimension_table;

-- Output (consistent snapshot at the start):
+----+-------+
| id | name  |
+----+-------+
| 1  | Row A |
| 2  | Row B |
| 3  | Row C |
```

**Data:**

```json
{
  "dimension_table": {
    "id": [1, 2, 3],
    "name": ["Row A", "Row B", "Row C"]
  }
}
```

**Flow:**

1. Yes
2. No
3. Start Transaction
4. Set Isolation Level
5. Read Dimension Table
6. Use Locking Mechanisms?
7. Acquire Locks
8. Continue with ETL
9. Perform ETL Operations
10. Commit Transaction

