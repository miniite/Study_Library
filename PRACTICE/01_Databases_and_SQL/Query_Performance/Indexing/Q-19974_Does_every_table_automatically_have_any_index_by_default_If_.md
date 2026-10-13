---
type: "practice"
difficulty: "Basic"
---

# Does every table automatically have any index by default? If so, which one?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ How Indexes Are Used](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/How_Indexes_Are_Used/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In database systems, whether a table has an index by default depends on the specific database management system (DBMS) you are using. Generally, not every table comes with any index automatically; however, some DBMSes may have certain default indices for metadata or internal purposes.

To address this question step-by-step from first principles:

Understanding Indexing in Databases: An index is a data structure that improves the speed of data retrieval operations on a database table at the cost of additional writes and storage space to maintain the index.

Default Indices Across DBMSes:

MySQL: MySQL creates an AUTO_INCREMENT primary key index by default for tables with auto-increment columns, but no other indices are automatically created unless explicitly defined in the schema.
PostgreSQL: PostgreSQL does not create any default indices. You need to define them manually or using a database migration tool.
SQL Server: SQL Server also doesn't create any default indices; you must define the necessary indices according to your query patterns and data access requirements.
Oracle: Oracle can automatically generate certain types of indices for tables, such as BITMAP indices for columns with low cardinality.

#### Example
Databases Without Default Indices:

If we consider PostgreSQL or SQL Server, no index is created by default. This means that every table you create will require manual indexing if you need to speed up queries.

Creating an Index: For example, in PostgreSQL, you would manually add a primary key or unique constraint, which implicitly creates an index:

SQL
CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(150)
);

In summary, not all databases automatically create indices for every table. Understanding the default behavior of your chosen database management system is crucial.

