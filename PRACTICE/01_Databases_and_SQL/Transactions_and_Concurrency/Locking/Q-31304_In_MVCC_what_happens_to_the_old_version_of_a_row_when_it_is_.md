---
type: "practice"
difficulty: "Deep"
---

# In MVCC, what happens to the old version of a row when it is updated?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ MVCC](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/Locking/MVCC.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

When you update a row in a system using Multi-Version Concurrency Control (MVCC), the old version of that row is retained and not directly modified. Instead, a new version of the row with updated values is created, leaving the old version intact.

#### Triaging the Question

The question asks about what happens to the old versions of rows when they are updated in MVCC systems. The interviewer is probing your understanding of how MVCC manages concurrent transactions and retains historical data without locking mechanisms that could block other operations. Assumptions include an understanding of transaction isolation levels, read consistency, and write durability.

#### First Principles Solution

Understanding MVCC:

MVCC is a concurrency control technique used in databases like PostgreSQL to ensure high performance under concurrent transactions.
It allows multiple versions of data to coexist by maintaining different snapshots or versions of the same row based on when they were read and written.

Row Versioning Mechanism:

When an update transaction modifies a row, it does not overwrite the current version of that row directly; instead, it creates a new version with updated values.
The old version remains unchanged and is stored in a different segment or table (often referred to as the "history" or "tombstone" table).

Transaction Isolation Levels:

Depending on the isolation level chosen (e.g., READ COMMITTED, REPEATABLE READ), readers can see either committed data only ("dirty reads") or no uncommitted data ("serializable").
Writers always see a consistent snapshot of the database at the start of their transaction and do not block other readers.

Handling Old Versions:

The old row versions are managed by the database system to avoid filling up storage with unnecessary historical data.
Periodic "vacuum" operations can reclaim space occupied by old, no longer needed versions.

Read-Consistency Example:

If a transaction starts and reads a version of a row at some point in time, it will continue to see that same version even if other transactions modify the row later.
This ensures that readers do not see partially committed data or "dirty" writes from concurrent transactions.

#### Closing Summary
& Common Mistakes:
Common Mistake: Assuming old versions are deleted immediately. In reality, they are retained but managed through periodic cleanup processes to maintain performance and storage efficiency.

#### Follow-Up
Question: How does MVCC handle space management for old row versions? Discuss the role of vacuuming and how it balances between retaining historical data and optimizing storage usage.
Depth + Language Match:

For a fresher, focus on basic concepts like snapshot isolation. For mid-level candidates, discuss trade-offs in transaction isolation levels. For senior roles, emphasize strategic considerations such as performance optimization and system design principles around concurrency control.

**Comparison:**

| id | name   | salary | _tx_id  | _valid_to         |
|----|--------|--------|---------|------------------|
| 1  | John Doe| 45000  | 123456  | null             |
| 1  | John Doe| 50000  | 987654  | now - interval '3h' |

**Code:**

```sql
-- Example of an MVCC update query in PostgreSQL
BEGIN;
UPDATE employees SET salary = 50000 WHERE id = 1 RETURNING *; -- Creates a new version with the updated salary

-- Vacuum operation to reclaim space from old versions (if needed)
VACUUM FULL employees;
```

**Example input / output:**

```text
Input:
BEGIN;
UPDATE employees SET salary = 50000 WHERE id = 1;

Output:
Row with ID 1 updated. A new version is created, and the old one remains in the history table.

Vacuum Operation (after some time):
VACUUM FULL employees;

Output: Old versions reclaimed for storage optimization.
```

**Data:**

```json
{
  "row_versions": [
    {
      "id": 1,
      "name": "John Doe",
      "salary": 45000,
      "_tx_id": 123456, // Transaction ID of the current version
      "_valid_to": null // Indicates this is the latest version until a newer one appears
    },
    {
      "id": 1,
      "name": "John Doe",
      "salary": 50000,
      "_tx_id": 987654, // Transaction ID of the previous version
      "_valid_to": now() - interval '3 hours' // Time when this version became obsolete
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Transaction Starts
4. Read Row Version
5. C
6. Update New Version
7. Vacuum Operation?
8. Reclaim Space
9. End Transaction

