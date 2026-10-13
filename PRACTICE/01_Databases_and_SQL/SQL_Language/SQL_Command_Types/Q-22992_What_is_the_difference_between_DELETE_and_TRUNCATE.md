---
type: "practice"
difficulty: "Intermediate"
---

# What is the difference between DELETE and TRUNCATE?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ DELETE vs TRUNCATE](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Command_Types/DELETE_vs_TRUNCATE.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In database development, understanding the differences between DELETE and TRUNCATE commands is crucial for efficient data management and performance optimization. Both commands are used to remove rows from a table, but they operate differently in terms of speed, transactional behavior, rollback capabilities, and impact on system resources.

#### Triage
What is Being Probed

The question aims to understand the nuances between DELETE and TRUNCATE, which are two common Data Manipulation Language (DML) commands. The interviewer might be probing your knowledge of how these operations affect transaction logs, performance, and whether they can be rolled back.

#### First-Principles Solution

Transaction Logs:

DELETE: Records each row deletion in the transaction log, making it slower compared to TRUNCATE.
TRUNCATE: Empties the table by deallocating the space used by the table's data pages and does not record individual row deletions, making it faster.

Rollback Capability:

DELETE: Can be rolled back if a transaction is active.
TRUNCATE: Cannot be rolled back once executed because no explicit log of deleted rows exists.

Performance Impact:

DELETE: Slower due to the transactional overhead and logging.
TRUNCATE: Faster as it bypasses the transactional overhead and logging process.

Data Integrity Constraints:

Both commands honor check constraints, meaning they will fail if a row violates a constraint defined on the table.

Row Count Information:

DELETE: Returns the number of rows affected by the operation.
TRUNCATE: Does not return any information about how many rows were deleted.

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, DELETE is used when you want to delete specific rows from a table with transactional control over each row. On the other hand, TRUNCATE is faster and more efficient for bulk deletions but lacks the ability to be rolled back once executed.

#### Common mistakes
include using DELETE unnecessarily in situations where TRUNCATE would be more appropriate, leading to slower performance and unnecessary use of transaction logs. Another mistake is assuming that both commands can be rolled back, which is not true for TRUNCATE.

Common Follow-Ups
How does a DELETE statement handle constraints compared to TRUNCATE?
In what scenarios would you choose one over the other?
What are the implications of using TRUNCATE on an index or a foreign key constraint?

**Code:**

```sql
-- Example of DELETE command
DELETE FROM Employees WHERE EmployeeID = 1;

-- Example of TRUNCATE command
TRUNCATE TABLE Employees;
```

