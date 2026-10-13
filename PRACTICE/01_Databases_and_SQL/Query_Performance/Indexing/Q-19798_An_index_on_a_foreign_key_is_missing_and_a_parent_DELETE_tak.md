---
type: "practice"
difficulty: "Intermediate"
---

# An index on a foreign key is missing and a parent DELETE takes a full-scan lock on the child. Walk through detecting the missing FK index and safely adding it under live load.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Adding Unique Indexes Safely](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Adding_Unique_Indexes_Safely.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of a parent DELETE operation taking a full-scan lock on the child table due to the missing foreign key (FK) index, we need to first detect the presence or absence of this index and then safely add it while minimizing performance impact during live operations. Here’s a step-by-step approach from first principles:

Understand the Problem: When you delete a parent record in a database with an FK relationship, without an appropriate index on the child table's foreign key column, the database engine might perform a full scan to ensure no orphaned records exist in the child table. This can lead to significant performance degradation during high-concurrency operations.

Identify the Missing Index: To detect whether this index is missing, we can use SQL queries and explain plans from our database management system (DBMS). Common DBMS like PostgreSQL or MySQL provide EXPLAIN commands that help us understand how queries are being executed.

Add the Index Safely Under Live Load: Adding an index under live load requires careful planning to avoid significant performance issues. We can use online DDL features if supported by our DBMS, which allow operations such as adding indexes without causing full table scans during the process.

#### Triaging the Question
The interviewer is probing your understanding of indexing and query optimization in databases.
Key assumptions: The database supports online DDL (like CREATE INDEX CONCURRENTLY in PostgreSQL).
You need to ensure minimal disruption during the index creation, which involves understanding live operations and transactional consistency.

#### First-Principles Solution

Check for the Missing Index:

Use an EXPLAIN command on typical DELETE queries from the parent table.
Look for full table scans or other signs that indicate a missing FK index is causing performance issues.

Use Online DDL to Add the Index:

If your DBMS supports CREATE INDEX CONCURRENTLY (PostgreSQL), use this feature to add the index without locking the table.
For other DBMS, consider using temporary indexes or partitioning strategies to minimize lock contention.

Monitor and Validate:

After adding the index, monitor performance metrics like query execution time, locking, and I/O operations.
Ensure that there are no new issues introduced by the index addition.

#### Closing Summary
+ Common Mistakes
Adding indexes can significantly improve query performance but must be done carefully to avoid impacting live operations.
Missing FK indices can lead to full table scans during critical operations like deletes, which can degrade performance.
Using online DDL features is crucial for minimizing downtime and ensuring data consistency while adding or modifying indexes.

#### Common Mistakes
Not Detecting the Issue Properly: Failing to use EXPLAIN plans to detect missing FK indices.
Ignoring Performance Impact: Not monitoring performance after adding an index, leading to unexpected degradation.
Poor Timing of Index Additions: Adding large indexes during peak hours without considering the impact.

**Code:**

```sql
-- Example SQL for detecting full table scans in PostgreSQL
EXPLAIN ANALYZE DELETE FROM parent_table WHERE foreign_key = 'some_value';
```

