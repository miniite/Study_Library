---
type: "practice"
difficulty: "Deep"
---

# You need to add a NOT NULL column with a default to a 400M-row table with zero downtime. How do you avoid a full-table rewrite and long metadata lock in PostgreSQL versus MySQL?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Check and NOT NULL Constraints](<../../../../01_Databases_and_SQL/Database_Design/Constraints_and_Keys/Check_and_NOT_NULL_Constraints.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When adding a NOT NULL column with a default value to a large table (400M rows) without causing downtime or long metadata locks, the choice of database system—PostgreSQL vs. MySQL—makes a significant difference. This problem involves understanding constraints and key management, as well as handling schema changes efficiently.

#### Triaging the Question

The core question here is about how to modify table structure with minimal impact on performance and availability. Specifically:

#### Cons
traints & Keys: Managing NOT NULL constraints.
Metadata Locks: Avoiding long metadata locks that can cause downtime during structural modifications.
Schema Changes: Implementing a strategy for adding columns without disrupting the existing data.

#### Assumptions
The table has existing data and requires no initial data migrations.
The column being added is NOT NULL with a default value to ensure all rows are populated automatically.
We need to minimize metadata locks to maintain high availability.
Step-by-Step Solution
PostgreSQL Approach

In PostgreSQL, adding a NOT NULL constraint without data migration can be achieved through the following steps:

Add Column Without Constraint:

SQL
ALTER TABLE your_table ADD COLUMN new_column datatype DEFAULT default_value;

This step adds the column but does not enforce the NOT NULL constraint.

Update Data with Default Value:

SQL
UPDATE your_table SET new_column = default_value;

Populate existing rows with the default value.

Add NOT NULL Constraint:

SQL
ALTER TABLE your_table ADD CONSTRAINT constraint_name CHECK (new_column IS NOT NULL);

This step ensures that all future inserts enforce NOT NULL.

MySQL Approach

In MySQL, adding a NOT NULL column with a default value is more complex due to its handling of schema changes and metadata locks. A common approach involves using an intermediate table or temporary column:

Create Intermediate Column:

SQL
ALTER TABLE your_table ADD COLUMN new_column datatype DEFAULT default_value;

Update Data Using Temporary Column (if needed, create a temporary column to hold the values temporarily):

SQL
UPDATE your_table SET temp_column = default_value WHERE new_column IS NULL;

If new_column is initially null, this step populates it with the default value.

Drop Temporary Column and Add NOT NULL Constraint:

SQL
ALTER TABLE your_table DROP COLUMN temp_column;  -- if used

-- Enforce NOT NULL constraint after ensuring all rows have a value
SET FOREIGN_KEY_CHECKS = 0;  -- Relax foreign key checks to avoid cascading changes
ALTER TABLE your_table MODIFY new_column datatype NOT NULL;
SET FOREIGN_KEY_CHECKS = 1;  -- Restore foreign key checks

#### Closing Summary

Both PostgreSQL and MySQL require careful planning when adding NOT NULL columns with default values. PostgreSQL offers a simpler path using the ALTER TABLE ADD COLUMN command directly, while MySQL requires intermediate steps to ensure data integrity without locking the entire table.

#### Common Mistakes
& Follow-Ups
Forgetting Default Value: Not setting a default value can lead to null columns.
Metadata Locks: Both databases have mechanisms that can cause long metadata locks. PostgreSQL's ALTER TABLE commands are generally less intrusive than MySQL's, but proper planning is crucial.
Data Inconsistency: Ensuring all data has the correct default value before adding constraints.

Always test your approach in a development environment to ensure robustness and efficiency.

**Code:**

```sql
-- PostgreSQL Example
ALTER TABLE your_table ADD COLUMN new_column datatype DEFAULT default_value;

-- MySQL Example
ALTER TABLE your_table ADD COLUMN new_column datatype DEFAULT default_value;

UPDATE your_table SET new_column = default_value WHERE new_column IS NULL;
```

