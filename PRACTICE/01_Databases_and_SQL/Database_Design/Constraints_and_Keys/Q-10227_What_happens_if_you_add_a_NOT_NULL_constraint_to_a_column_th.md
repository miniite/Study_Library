---
type: "practice"
difficulty: "Intermediate"
---

# What happens if you add a NOT NULL constraint to a column that already contains NULL values?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Check and NOT NULL Constraints](<../../../../01_Databases_and_SQL/Database_Design/Constraints_and_Keys/Check_and_NOT_NULL_Constraints.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When adding a NOT NULL constraint to a column in an existing table, any rows with NULL values in this column will violate the new constraint, leading to either data integrity issues or errors during the DDL execution.

#### Triage
and Assumptions:
We are working on an existing database schema.
The specific column already contains NULL values.
The SQL dialect being used is standard (e.g., MySQL, PostgreSQL).

Adding a NOT NULL constraint means that every row in this column must contain a non-null value. If there are rows with NULL, these rows will not meet the new requirement and will need to be addressed.

#### First Principles Solution
Identify Rows: Determine which rows have NULL values.
Update Rows: Update or delete these rows as necessary.
Add Constraint: Apply the NOT NULL constraint to the column.
Step-by-Step Solution:

Check Current State:

SQL
SELECT * FROM table_name WHERE column_name IS NULL;

This query will identify all rows where the specified column has NULL values.

Update Rows with Non-null Values (if possible):

If there are acceptable non-null default values, update these rows.

#### Example
SQL
UPDATE table_name SET column_name = 'default_value' WHERE column_name IS NULL;

Delete Rows if Acceptable:

If the NULL values cannot be replaced and must be removed, delete those rows.

#### Example
SQL
DELETE FROM table_name WHERE column_name IS NULL;

Add NOT NULL Constraint:

Apply the constraint to the column.

#### Example
(PostgreSQL):
SQL
ALTER TABLE table_name ADD CONSTRAINT constraint_name CHECK (column_name IS NOT NULL);

#### Closing Summary
and Common Mistakes/Follow-ups:
Common Mistake: Directly adding a NOT NULL constraint without handling existing NULL values can cause the operation to fail.

#### Follow-Up
Ensuring that data quality is maintained, especially in large tables. Use of temporary constraints or transactions might be necessary for complex databases.

#### Closing Summary

Adding a NOT NULL constraint to a column that already contains NULL values requires identifying and addressing these nulls before the constraint can be applied successfully. The process involves checking current state, updating or deleting rows, and then applying the new constraint.

