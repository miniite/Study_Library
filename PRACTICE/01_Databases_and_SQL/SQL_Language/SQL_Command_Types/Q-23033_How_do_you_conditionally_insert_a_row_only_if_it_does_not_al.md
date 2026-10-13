---
type: "practice"
difficulty: "Intermediate"
---

# How do you conditionally insert a row only if it does not already exist?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Upserts](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Upserts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To conditionally insert a row into a database table only if it does not already exist, we need to use a combination of SQL commands and ensure proper handling of concurrent operations. This task involves understanding DML (Data Manipulation Language) with INSERT statements and leveraging DCL (Data Control Language) through transactional control (BEGIN, COMMIT, or ROLLBACK) and DDL (Data Definition Language) if necessary for schema modifications.

First, let's break down the problem. We need to check if a row already exists before attempting an insertion. If it does not exist, we insert the new row; otherwise, no action is taken. This scenario can be handled using INSERT ... ON DUPLICATE KEY UPDATE (MySQL), MERGE INTO (Oracle), or common subqueries and transactions in other SQL dialects.

#### Triaging the Question

The interviewer might be probing your understanding of:

SQL Constructs: Familiarity with DML commands like INSERT.
Data Control Mechanisms: Knowledge of how to handle concurrent operations.
Transaction Management: Understanding of transactional integrity and rollbacks.

#### Solution
from First Principles
Step 1: Define the Table

Assume we have a table named employees with columns id, name, age.

SQL
CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR(255),
    age INT
);
Step 2: Using INSERT ... ON DUPLICATE KEY UPDATE

This is the simplest and most straightforward approach for MySQL.

#### Example
Input:

Check if a row with id = 1 and name = 'Alice' exists.
If not, insert the new record.
SQL
BEGIN;
INSERT INTO employees (id, name, age) VALUES (1, 'Alice', 30)
ON DUPLICATE KEY UPDATE id=id; -- This update is a no-op to avoid changing existing rows
COMMIT;
Step 3: Using Common Subquery in Other SQL Dialects

For databases that do not support INSERT ... ON DUPLICATE KEY UPDATE, you can use a common subquery.

#### Example
Input:

Check if the row does not exist.
If it doesn't, insert the new record.
SQL
BEGIN;
IF NOT EXISTS (SELECT 1 FROM employees WHERE id = 1 AND name = 'Alice') THEN
    INSERT INTO employees (id, name, age) VALUES (1, 'Alice', 30);
END IF;
COMMIT;
Step 4: Using MERGE INTO in Oracle

Oracle provides the MERGE INTO statement for this purpose.

#### Example
Input:

Merge or insert based on a condition.
SQL
BEGIN
    MERGE INTO employees e
    USING (SELECT 1 AS id, 'Alice' AS name, 30 AS age FROM DUAL) src
    ON (e.id = src.id AND e.name = src.name)
    WHEN NOT MATCHED THEN INSERT VALUES(src.id, src.name, src.age);
COMMIT;

#### Closing Summary

To conditionally insert a row only if it does not already exist, you can use INSERT ... ON DUPLICATE KEY UPDATE in MySQL or equivalent constructs in other SQL dialects. For more complex scenarios, consider using common subqueries or MERGE INTO. Always ensure transactional integrity by managing BEGIN and COMMIT statements.

#### Common Mistakes
Forgetting to start a transaction: This can lead to inconsistent data if the operation fails.
Using incorrect conditions in the subquery: Ensure your condition correctly identifies whether the row should be inserted or updated.
Misusing ON DUPLICATE KEY UPDATE: Be careful not to inadvertently update existing rows, as this might introduce unintended side effects.

**Comparison:**

| id | name   | age |
|----|--------|-----|
|  1 | Alice  |  30 |
|  2 | Charlie| 35  |

**Code:**

```sql
-- MySQL Example
BEGIN;
INSERT INTO employees (id, name, age) VALUES (1, 'Alice', 30)
ON DUPLICATE KEY UPDATE id=id; -- No-op

-- Oracle Example
MERGE INTO employees e
USING (SELECT 1 AS id, 'Alice' AS name, 30 AS age FROM DUAL) src
ON (e.id = src.id AND e.name = src.name)
WHEN NOT MATCHED THEN INSERT VALUES(src.id, src.name, src.age);

-- Common Subquery Example
IF NOT EXISTS (SELECT 1 FROM employees WHERE id = 1 AND name = 'Alice') THEN
    INSERT INTO employees (id, name, age) VALUES (1, 'Alice', 30);
END IF;
```

**Example input / output:**

```text
BEGIN;
-- Check and insert if not exists in MySQL
IF NOT EXISTS (SELECT 1 FROM employees WHERE id = 1 AND name = 'Alice') THEN
    INSERT INTO employees (id, name, age) VALUES (1, 'Alice', 30);
END IF;

COMMIT;
```

**Data:**

```sql
CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR(255),
    age INT
);
INSERT INTO employees (id, name, age) VALUES (1, 'Bob', 40), (2, 'Charlie', 35);
```

**Flow:**

1. No
2. Yes
3. Check if row exists
4. Row Exists?
5. Insert new row
6. Transaction Complete

