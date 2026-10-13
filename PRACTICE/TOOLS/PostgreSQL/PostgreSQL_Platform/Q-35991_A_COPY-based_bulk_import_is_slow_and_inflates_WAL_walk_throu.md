---
type: "practice"
difficulty: "Deep"
---

# A COPY-based bulk import is slow and inflates WAL; walk through disabling indexes/constraints, using UNLOGGED tables, and the durability trade-offs of each speedup.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Materialized Views vs Incremental Aggregates](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Materialized_Views_vs_Incremental_Aggregates.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of a slow copy-based bulk import in PostgreSQL that inflates the Write-Ahead Log (WAL), we can consider several strategies to improve performance. These include disabling indexes and constraints, using UNLOGGED tables, and understanding the trade-offs involved.

#### Triage

The question is asking for an explanation on how to optimize a copy-based bulk import in PostgreSQL by disabling certain features that log changes to the WAL (Write-Ahead Log). The goal is to increase speed but understand the potential durability impacts. We need to consider the impact of disabling indexes and constraints, as well as using UNLOGGED tables.

#### First-Principles Solution

Disabling Indexes/Constraints:

Disabling indexes or constraints can significantly reduce the time required for bulk imports since these features add overhead during the insertion process.

Using UNLOGGED Tables:

UNLOGGED tables are designed to sacrifice durability for speed, as they do not log changes to the WAL.

Understanding Trade-Offs:

Disabling indexes or constraints means that subsequent queries may be slower since those indexes and constraints would need to be rebuilt.
Using UNLOGGED tables can result in data loss if the database crashes during operations, as these tables do not write to the WAL.
Detailed Steps

Disabling Indexes:

Before importing, disable all indexes on the table using ALTER TABLE table_name DISABLE INDEXES.
After the import is complete, re-enable the indexes with ALTER TABLE table_name ENABLE INDEXES.

Disabling Constraints (Primary Keys/Foreign Keys):

Similarly, you can disable constraints to speed up imports.
Disable primary keys or foreign key constraints before importing using ALTER TABLE table_name DISABLE CONSTRAINTS.
Re-enable these constraints after the import with ALTER TABLE table_name ENABLE CONSTRAINTS.

Using UNLOGGED Tables:

Create a new UNLOGGED table and copy data into it.

#### Example
SQL:
SQL
CREATE UNLOGGED TABLE tmp_table AS SELECT * FROM original_table;

#### Closing Summary
+ Common Mistakes/Follow-Ups

Disabling indexes, constraints, and using UNLOGGED tables can significantly speed up bulk imports but come with durability trade-offs. Always ensure that the data integrity requirements are met before applying these optimizations.

#### Common mistakes
include:

Not enabling back the constraints after importing.
Overlooking the potential for data loss if using UNLOGGED tables without a backup strategy.
Failing to monitor performance post-import, especially when rebuilding indexes or constraints.

#### Follow-ups
might involve discussing how to balance performance and durability in a production environment or exploring other bulk import methods like COPY with streaming.

**Code:**

```sql
-- Example disabling and enabling indexes
ALTER TABLE table_name DISABLE INDEXES;
-- Import data here
ALTER TABLE table_name ENABLE INDEXES;

-- Disabling constraints before import
ALTER TABLE table_name DISABLE CONSTRAINTS;
-- Import data here
ALTER TABLE table_name ENABLE CONSTRAINTS;
```

