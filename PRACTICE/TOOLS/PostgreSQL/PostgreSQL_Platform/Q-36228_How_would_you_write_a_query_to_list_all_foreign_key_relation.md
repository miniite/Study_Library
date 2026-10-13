---
type: "practice"
difficulty: "Intermediate"
---

# How would you write a query to list all foreign key relationships in a database?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Many-to-many Modeling](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Many-to-many_Modeling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To list all foreign key relationships in a PostgreSQL database, we need to examine the system catalogs that store metadata about the schema and constraints. This is a common task when dealing with complex schemas or performing database audits.

#### Triage

The core of this problem involves understanding how PostgreSQL stores its relational data structures, specifically focusing on foreign keys (constraints). The interviewer is probing your knowledge of PostgreSQL's internal schema management and SQL querying skills. We will assume that the goal is to retrieve a list of all foreign key relationships across tables in the database.

#### First-Principles Solution
Identify Relevant Tables: PostgreSQL stores constraint information in several system catalogs, such as pg_constraint and information_schema.table_constraints.
Query Constraints: Use SQL queries to join these tables to get a list of all foreign keys.
Filter Foreign Keys: Ensure that only foreign key constraints are returned.

Here’s the step-by-step solution:

Step 1: Query pg_constraint

The pg_constraint table contains information about various types of constraints, including foreign keys. We can use this table to find all constraint details.

SQL
SELECT
    conname AS constraint_name,
    conrelid::regclass AS parent_table,
    confrelid::regclass AS child_table,
    conkey AS parent_columns,
    confkey AS child_columns,
    contype
FROM
    pg_constraint
WHERE
    contype = 'f';

This query filters for only those constraints (contype = 'f') that are foreign keys.

Step 2: Join with information_schema.table_constraints

Alternatively, we can use the information_schema views to get more readable and detailed information:

SQL
SELECT
    tc.constraint_name,
    tc.table_name AS parent_table,
    kcu.column_name AS parent_columns,
    ccu.table_name AS child_table,
    ccu.column_name AS child_columns
FROM
    information_schema.table_constraints tc
JOIN
    information_schema.key_column_usage kcu ON
        tc.constraint_catalog = kcu.constraint_catalog
        AND tc.constraint_schema = kcu.constraint_schema
        AND tc.constraint_name = kcu.constraint_name
JOIN
    information_schema.constraint_column_usage ccu ON
        tc.constraint_catalog = ccu.constraint_catalog
        AND tc.constraint_schema = ccu.constraint_schema
        AND tc.constraint_name = ccuconstraint_name
WHERE
    tc.constraint_type = 'FOREIGN KEY';

This query joins the relevant views to provide a more human-readable output.

Step 3: Combine Results (Optional)

If you want to combine both results, you can use UNION ALL:

SQL
SELECT * FROM (
    SELECT
        conname AS constraint_name,
        conrelid::regclass AS parent_table,
        confrelid::regclass AS child_table,
        conkey AS parent_columns,
        confkey AS child_columns,
        contype
    FROM
        pg_constraint
    WHERE
        contype = 'f'
) AS foreign_keys

UNION ALL

SELECT
    tc.constraint_name,
    tc.table_name AS parent_table,
    kcu.column_name AS parent_columns,
    ccu.table_name AS child_table,
    ccu.column_name AS child_columns
FROM
    information_schema.table_constraints tc
JOIN
    information_schema.key_column_usage kcu ON
        tc.constraint_catalog = kcu.constraint_catalog
        AND tc.constraint_schema = kcu.constraint_schema
        AND tc.constraint_name = kcu.constraint_name
JOIN
    information_schema.constraint_column_usage ccu ON
        tc.constraint_catalog = ccu.constraint_catalog
        AND tc.constraint_schema = ccu.constraint_schema
        AND tc.constraint_name = ccu.constraint_name
WHERE
    tc.constraint_type = 'FOREIGN KEY';

#### Closing Summary
+ Common Mistakes/Follow-Ups

The solution above provides a comprehensive way to list all foreign key relationships in a PostgreSQL database. The primary mistakes include not correctly filtering for contype or missing the join conditions between tables.

#### Follow-Ups

How would you handle databases with very large schemas?
Can you modify this query to also show the referenced columns from the parent table?
What if there are circular foreign key relationships? How can you detect and handle them?

**Code:**

```sql
-- Query for pg_constraint
SELECT
    conname AS constraint_name,
    conrelid::regclass AS parent_table,
    confrelid::regclass AS child_table,
    conkey AS parent_columns,
    confkey AS child_columns,
    contype
FROM
    pg_constraint
WHERE
    contype = 'f';

-- Query for information_schema views
SELECT
    tc.constraint_name,
    tc.table_name AS parent_table,
    kcu.column_name AS parent_columns,
    ccu.table_name AS child_table,
    ccu.column_name AS child_columns
FROM
    information_schema.table_constraints tc
JOIN
    information_schema.key_column_usage kcu ON
        tc.constraint_catalog = kcu.constraint_catalog
        AND tc.constraint_schema = kcu.constraint_schema
        AND tc.constraint_name = kcu.constraint_name
JOIN
    information_schema.constraint_column_usage ccu ON
        tc.constraint_catalog = ccu.constraint_catalog
        AND tc.constraint_schema = ccu.constraint_schema
        AND tc.constraint_name = ccu.constraint_name
WHERE
    tc.constraint_type = 'FOREIGN KEY';
```

