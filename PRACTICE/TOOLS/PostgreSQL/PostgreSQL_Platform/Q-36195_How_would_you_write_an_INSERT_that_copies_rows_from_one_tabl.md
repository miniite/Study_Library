---
type: "practice"
difficulty: "Intermediate"
---

# How would you write an INSERT that copies rows from one table into another using INSERT INTO ... SELECT?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ RETURNING Clause](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/RETURNING_Clause.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To copy rows from one table to another in PostgreSQL, we use the INSERT INTO ... SELECT statement. This is a fundamental technique to move data between tables without manually specifying each column or row. The core idea here is to select all columns and rows from the source table and insert them into the target table.

#### Triage
What the Question Is Asking

The question is asking you to describe how to write an SQL statement that copies rows from one PostgreSQL table (the source) into another table (the destination). This involves selecting data from a source table and inserting it into a target table, ensuring that all columns match or have appropriate default values.

Step-by-Step Solution
Identify Source and Target Tables: Determine the names of the tables you want to copy data from and to.
Check Column Matching: Ensure that the columns in both tables either match exactly or can be implicitly cast to each other, especially if there are differing types or constraints on the target table.
Write the INSERT INTO ... SELECT Statement: Use this syntax:
SQL
INSERT INTO target_table (column1, column2, ...)
SELECT source_table.column1, source_table.column2, ...
FROM source_table;
Handle Potential Issues: Consider issues like foreign key constraints and ensure that the data being copied does not violate any constraints in the destination table.
Sample Code
SQL
-- Example: Copying rows from 'employees' to 'backup_employees'
INSERT INTO backup_employees (id, name, department_id, salary)
SELECT id, name, department_id, salary
FROM employees;

#### Closing Summary
and Common Mistakes/Follow-Ups

This INSERT INTO ... SELECT statement is a powerful tool in PostgreSQL. However, there are common pitfalls to avoid:

Column Mismatch: Ensure that the columns in both tables match or can be implicitly cast.

#### Cons
traints Violation: Be cautious of foreign key constraints and unique constraints on the target table.
Data Types: Make sure data types align between source and target columns.

#### Common Mistakes
Ignoring Column Names: Failing to list column names explicitly in both INSERT INTO and SELECT can lead to errors, especially if there are unnamed or default generated columns.
Overlooking Constraints: Copying data into a table with constraints (like unique keys) might violate them unless the source data already meets these criteria.
Final Thoughts

When using this method, always validate the schema of both tables and ensure that you understand the implications of the copy operation. This approach is efficient for simple data migrations but may require additional steps if dealing with complex schemas or large datasets.

**Code:**

```sql
INSERT INTO backup_employees (id, name, department_id, salary)
SELECT id, name, department_id, salary
FROM employees;
```

