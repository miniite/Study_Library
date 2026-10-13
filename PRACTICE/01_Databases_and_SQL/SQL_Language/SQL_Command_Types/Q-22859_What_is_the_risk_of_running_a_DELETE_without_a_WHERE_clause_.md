---
type: "practice"
difficulty: "Intermediate"
---

# What is the risk of running a DELETE without a WHERE clause and how do you avoid it?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ DROP and CASCADE](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Command_Types/DROP_and_CASCADE.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Deleting rows from a table without specifying any conditions can have severe consequences, as all rows in the table will be removed. This is a common mistake that can lead to data loss or incorrect data integrity. To avoid such risks, we should always use a WHERE clause when performing deletions.

To understand this better, let's break down the problem from first principles.

**Code:**

```sql
-- Example Table
CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    department VARCHAR(50)
);
```

**Example input / output:**

```text
Input:
DELETE FROM employees;

Expected Output:
All rows in the `employees` table will be deleted.

Actual Output (if no WHERE clause is used):
- The entire `employees` table becomes empty.
- No rows are left, and the operation can't be undone easily without a backup or rollback mechanism.
```

