---
type: "practice"
difficulty: "Deep"
---

# You must add an index to a large production source table used for extraction. How would you do it without blocking the running pipeline?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Index Tuning in Practice](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Index_Tuning_in_Practice/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of adding an index to a large production source table while ensuring minimal impact on the running extraction pipeline, we need to carefully plan and execute our steps from first principles. In this context, "minimal impact" means we want to avoid blocking or significantly slowing down the existing data processing tasks.

TL;DR

We will use a temporary table, parallel indexing, and a strategic schema design to add an index without disrupting the ongoing pipeline.

#### Triaging the Question

The core of the question involves balancing performance optimization with operational stability. The interviewer is probing your understanding of database operations, concurrency management, and practical strategies for maintaining system integrity during changes. Key assumptions include:

We are working within a relational database (e.g., PostgreSQL, MySQL).
The production pipeline runs in parallel with read/write operations on the table.
Our goal is to minimize downtime or performance degradation.
Step-by-Step Solution from First Principles

Identify the Necessary Index: Determine which columns should be indexed based on query patterns and performance bottlenecks. For instance, if most queries filter by customer_id and timestamp, we might add an index to (customer_id, timestamp).

Create a Temporary Table: Instead of modifying the production table directly, create a temporary table that mirrors the schema of the source table but with the new index added initially. This allows us to test the index without impacting live data or ongoing operations.

SQL
CREATE TABLE temp_table AS SELECT * FROM production_table;
CREATE INDEX idx_temp ON temp_table (customer_id, timestamp);

Parallel Indexing: Use parallel execution capabilities of your database system to speed up the indexing process. This can significantly reduce the time taken for adding an index, especially on large tables.

SQL
-- For PostgreSQL
CREATE INDEX CONCURRENTLY idx_temp ON temp_table (customer_id, timestamp);

Monitor and Validate: Ensure that the new index is functioning as expected by running some sample queries against both temp_table and the production table to confirm performance improvements.

Migrate Data and Schema: Once validation is complete, perform a schema migration to replace the old table with the indexed version.

SQL
-- For PostgreSQL (assuming we use CREATE TABLE AS for the final step)
DROP TABLE production_table;
ALTER TABLE temp_table RENAME TO production_table;

Cleanup: Remove any unnecessary temporary tables or indexes that were used during this process to keep your database clean.

#### Closing Summary
and Common Mistakes

By following these steps, we ensure that the index addition does not block the running pipeline and minimizes disruptions. Key takeaways include:

Using a temporary table for testing and development.
Leveraging parallel indexing techniques.
Carefully monitoring performance before making changes live.

Common pitfalls include failing to test in a staging environment first or overlooking concurrency issues during the switch-over phase.

#### Common Mistakes/Follow-Ups
Failing to Test in Staging: Always validate on a non-production copy of your database to ensure everything works as expected.
Insufficient Monitoring: Not having proper monitoring and logging set up can lead to unforeseen issues post-deployment.
Ignoring Concurrency Issues: Not handling locking or blocking during the migration phase could cause performance degradation.

**Code:**

```sql
-- Example SQL for creating a temporary table and indexing it concurrently
CREATE TABLE temp_table AS SELECT * FROM production_table;
CREATE INDEX CONCURRENTLY idx_temp ON temp_table (customer_id, timestamp);
```

**Flow:**

1. Start
2. Create Temp Table
3. Mirror Schema
4. Create Index Concurrently
5. Monitor & Validate
6. Perform Migration
7. Cleanup Staging Data
8. End

