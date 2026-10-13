---
type: "practice"
difficulty: "Basic"
---

# How do you view all constraints defined on a table in your chosen RDBMS?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Check and NOT NULL Constraints](<../../../../01_Databases_and_SQL/Database_Design/Constraints_and_Keys/Check_and_NOT_NULL_Constraints.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To view all constraints defined on a table in an RDBMS, you need to understand how metadata is stored and accessed within the database system. In this context, we will focus on PostgreSQL as our chosen RDBMS, though similar methods apply to other databases like MySQL or SQL Server with slight variations.

#### Triage
What’s Being Probed

The question aims to test your understanding of how relational databases manage schema information and your ability to query this information using SQL. The interviewer is likely looking for familiarity with database metadata, common querying techniques, and possibly some nuances in the specific RDBMS (PostgreSQL here).

#### First-Principles Solution

To view all constraints defined on a table, you can query the system catalogs or information schema, which store detailed information about the tables, columns, indexes, and constraints. For PostgreSQL, we will use information_schema to fetch this data.

Step-by-Step Solution:

Identify Relevant Tables: The key tables in information_schema that contain constraint information are:

#### cons
traints: Stores definitions of table constraints.
foreign_tables, referenced_tables, and related views for foreign key (FK) relationships.

Query Constraints:

SQL
SELECT

#### cons
traint_name,
    table_schema,
    table_name,

#### cons
traint_type,
    is_deferrable,
    deferrable_initially
FROM
    information_schema.table_constraints
WHERE
    table_schema = 'public' AND  -- Modify schema name as needed
    table_name = 'your_table';   -- Replace with your actual table name

Query Foreign Keys:

SQL
SELECT

#### cons
traint_name,

#### cons
traint_column_names,
    referenced_table_schema,
    referenced_table_name,
    referenced_column_names
FROM
    information_schema.constraint_column_usage c
INNER JOIN
    information_schema.table_constraints t ON c.constraint_catalog = t.constraint_catalog
                                            AND c.constraint_schema = t.constraint_schema
                                            AND c.constraint_name = t.constraint_name
WHERE

#### cons
traint_type = 'FOREIGN KEY'
    AND table_schema = 'public'    -- Modify schema name as needed
    AND table_name = 'your_table'; -- Replace with your actual table name

#### Closing Summary
and Common Mistakes/Follow-Ups

By executing these queries, you can retrieve all the constraints (both primary keys and foreign keys) defined on a specific table. The first query will give you a comprehensive list of all kinds of constraints, while the second one focuses specifically on foreign key relationships.

Common Pitfalls:

Forgetting to specify the correct schema name.
Misunderstanding the difference between information_schema views and system catalogs like pg_constraint in PostgreSQL.
Not considering other types of constraints beyond primary keys (e.g., unique, check).

If asked further, you might explore how to handle different RDBMSes or discuss performance implications for larger databases.

