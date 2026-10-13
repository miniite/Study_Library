---
type: "practice"
difficulty: "Deep"
---

# What is a row-level lock versus a table-level lock in Oracle and when does Oracle escalate locking?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Oracle Isolation and Read Consistency](<../../../../TOOLS/Oracle/Oracle_Platform/Oracle_Isolation_and_Read_Consistency.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In Oracle, row-level locks and table-level locks are fundamental concepts used to manage concurrency and data consistency during transactions. A row-level lock confines the lock on a single row, while a table-level lock secures all rows within a table. Oracle's locking mechanism automatically decides when to escalate from row-level to table-level based on the nature of the operation and system load.

**Comparison:**

| Lock Type | Scope         | Examples of Operations |
|-----------|---------------|-----------------------|
| Row-Level  | Single row     | SELECT ... FOR UPDATE, DELETE FROM ... WHERE ... |
| Table-Level| All rows       | None (automatically managed) |

**Code:**

```text
-- Example of an explicit row-level lock
SELECT * FROM employees WHERE employee_id = 102 FOR UPDATE;

-- Table-level lock (rarely used in practice as Oracle manages it automatically)
LOCK TABLE employees IN SHARE MODE;
```

