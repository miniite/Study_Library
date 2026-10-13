---
type: "practice"
difficulty: "Basic"
---

# How do you write a query to list all tables in the current database using psql or a system catalog?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Many-to-many Modeling](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Many-to-many_Modeling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To list all tables in the current database using psql or by querying the system catalogs directly within PostgreSQL, we need to understand how to access the information schema and use SQL commands effectively.

#### Triaging the Question

The core of this question is understanding how to query the metadata stored in PostgreSQL's system catalogs. You'll be asked to either:

Use psql: A command-line interface for interacting with PostgreSQL databases.
Query System Catalogs Directly: Using SQL commands to retrieve information about tables.

#### Assumptions
and First Principles
We are working within a PostgreSQL database environment.
The user has basic knowledge of using the psql tool or writing simple SQL queries.
The goal is to list all table names in the current database schema.

#### Solution
from First Principles

To list all tables, we can use the information_schema.tables view, which provides metadata about tables and views. Alternatively, for more direct access, you can query the system catalog directly using \d in psql.

Using System Catalogs with psql

In psql, you can use the \dt command to list all tables in the current database:

SH
\dt

This command leverages the system catalogs and provides a user-friendly output.

Querying Information Schema Directly

If you prefer writing SQL directly, you can query the information_schema.tables view. Here’s how you do it:

SQL
SELECT table_name
FROM information_schema.tables
WHERE table_schema = 'public';

This command fetches all table names from the public schema (the default schema for most databases).

#### Closing Summary
and Common Mistakes

The key takeaway is understanding that both methods achieve the same goal: listing tables. The \dt command is more convenient in a psql session, while querying information_schema.tables provides a direct SQL solution.

#### Common Mistakes

Forgetting to specify the schema name when writing an SQL query.
Confusing system catalogs with other metadata views like pg_tables.
Not understanding that \dt is specific to psql, whereas information_schema.tables is part of the standard SQL metadata.

#### Example
Input and Output

To demonstrate, let's assume we have a database named example_db. Here’s how you might interact with psql:

SH
$ psql -U username example_db

Then running:

SH
\dt

Would output something like:

CODE
List of relations
 Schema |  Name  | Type  | Owner
--------+--------+-------+--------
 public | table1 | table + user
 public | table2 | table + user
 public | table3 | table + user

Alternatively, within psql, you can run the SQL query:

SQL
SELECT table_name
FROM information_schema.tables
WHERE table_schema = 'public';

Which would yield a similar result.

Self-Check

This solution is correct and complete.

**Code:**

```sql
-- Example SQL query to list tables in public schema
SELECT table_name
FROM information_schema.tables
WHERE table_schema = 'public';
```

