---
type: "practice"
difficulty: "Basic"
---

# Where is the definition of a stored procedure stored in the database?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Stored Procedure Fundamentals](<../../../../01_Databases_and_SQL/Database_Objects/Stored_Procedures/Stored_Procedure_Fundamentals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Sure, let's dive into this basic but important question about where the definition of a stored procedure is stored in a database.

TL;DR:

The definition of a stored procedure is typically stored within the system catalog or metadata tables of the database management system (DBMS). This information is critical for the DBMS to understand and execute the procedures. For example, in PostgreSQL, this data can be found in tables like pg_proc.

#### Triaging the Question

When asked where a stored procedure's definition is stored, the interviewer is probing your understanding of how databases manage metadata and how stored procedures are implemented at the database level.

Step-by-Step Solution from First Principles:
Stored Procedure Definition:
A stored procedure is essentially a sequence of SQL statements bundled together with a name, which can be executed as a single unit.
Metadata Storage:
Database management systems need to store metadata about these procedures for various purposes like execution, monitoring, and maintenance.
System Catalogs:
Most DBMSes have a system catalog or metadata schema where this information is stored.

For example, in the PostgreSQL ecosystem:

The pg_proc table contains definitions of all the stored procedures.
Each row in pg_proc represents one procedure, storing details such as the name, body (the actual SQL), and other relevant attributes like owner and language used.

#### Example
PostgreSQL’s pg_proc Table
SQL
-- Sample output from pg_proc for a hypothetical procedure
SELECT proname, pronamespace, prosrc
FROM pg_proc
WHERE proname = 'my_sample_procedure';

This query would return the definition of a stored procedure named my_sample_procedure.

#### Closing Summary
and Common Mistakes:

Understanding that the definitions are stored in system catalogs is key. A common mistake beginners might make is assuming this information is only accessible by the database administrator or via complex queries. In reality, these details are typically readily available through standard SQL queries on the DBMS's own metadata schema.

Self-Check:

This response covers the basics and provides an example relevant to PostgreSQL, ensuring clarity and completeness.

**Comparison:**

| Column Name | Description |
|-------------|-------------|
| proname     | The name of the stored procedure. |
| pronamespace | The namespace (schema) where the procedure resides. |
| prosrc      | The source code of the stored procedure in SQL. |

**Code:**

```sql
-- Sample query to retrieve a stored procedure definition in PostgreSQL
SELECT proname, pronamespace, prosrc
FROM pg_proc
WHERE proname = 'my_sample_procedure';
```

