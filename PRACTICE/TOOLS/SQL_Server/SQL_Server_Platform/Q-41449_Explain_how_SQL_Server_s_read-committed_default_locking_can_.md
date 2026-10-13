---
type: "practice"
difficulty: "Expert"
---

# Explain how SQL Server's read-committed default (locking) can cause a query to read the same row twice or skip rows during an index scan while data moves.

**Difficulty:** Expert · **Levels:** SDE-1, SDE-1/2 · [↑ Read Committed Locking Anomalies](<../../../../TOOLS/SQL_Server/SQL_Server_Platform/Read_Committed_Locking_Anomalies.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with SQL Server, understanding its locking behavior is crucial for writing efficient and predictable queries. The READ_COMMITTED isolation level, which is the default in most databases, uses row-level locks to prevent dirty reads. However, this can sometimes lead to unexpected behaviors like reading the same row twice or skipping rows during an index scan when data moves.

Let's break down these scenarios step by step:

Understanding Row-Level Locking:

In READ_COMMITTED isolation, each transaction is granted a read lock on any rows it reads until the end of the transaction.
This ensures that another transaction cannot modify those rows during the current transaction.

Scenario 1: Reading the Same Row Twice:

#### Cons
ider two transactions, T1 and T2, both reading from an index that is being updated by a third transaction, T3.
When T1 starts, it reads row R1. Then, T3 updates row R1 while T1 is still running but hasn’t released its read lock on R1.
When T2 starts and tries to read the same row R1 (which has changed due to T3's update), SQL Server may allow T2 to read the old state of R1 based on when it started, causing a conflict.

Scenario 2: Skipped Rows During Index Scan:

Suppose an index is being rebuilt or updated by a background process (e.g., maintenance tasks).
A transaction scanning the index may start at a point where part of the index has been updated.
If the transaction reads from a state before the update was fully committed, it might skip over rows that have already been updated.

Why This Happens:

READ_COMMITTED uses optimistic locking; each transaction checks for conflicts when committing changes.
Between the time a row is read and the end of the transaction, other transactions can modify the data, leading to inconsistencies in reads.

Mitigation Strategies:

Use appropriate isolation levels (e.g., SNAPSHOT, which uses snapshot isolation).
Implement application-level logic to handle concurrency issues.
Ensure that index maintenance tasks are scheduled during off-peak times or use parallelism management techniques.

**Comparison:**

| ID | Value    |
|----|----------|
| 1  | OldValue |
| 2  | NewValue |

**Code:**

```sql
-- Example of reading the same row twice with READ_COMMITTED isolation
BEGIN TRANSACTION;

SELECT * FROM MyTable WHERE Id = 1 FOR UPDATE;

-- T2 could read an old state here
SELECT * FROM MyTable WHERE Id = 1;
```

**Example input / output:**

| Input | Output (T1) | Output (T2) |
|-------|-------------|-------------|
| `SELECT * FROM MyTable WHERE Id = 1;` in T1 starting state | `{ "Id": 1, "Value": "OldValue" }` (read from old state) |
| After `UPDATE MyTable SET Value = 'NewValue' WHERE Id = 1;` by T3 | `{ "Id": 1, "Value": "NewValue" }` (T2 reads the updated state after commit) |

**Data:**

```json
{
    "MyTable": [
        { "Id": 1, "Value": "OldValue" },
        { "Id": 2, "Value": "NewValue" }
    ]
}
```

**Flow:**

1. FLOWCHART · MERMAID
2. graph TD;
3. A[Transaction T1 starts] --> B{Read Row R1};
4. B --> C(T3 updates R1);
5. C --> D{Transaction T2 starts};
6. D --> E{T2 reads R1?};
7. E --> F(Row is read from old state) | G(Row is read from new state after commit);
8. F --> H[Conflict detected] | I[No conflict, consistent read];
9. I --> J[End of transaction]

