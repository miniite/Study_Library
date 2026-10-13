---
type: "practice"
difficulty: "Intermediate"
---

# How would you safely drop a column that you *think* is unused on a 200M-row table, and how do you prove nothing reads it before the destructive migration?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Online Schema Changes](<../../../../TOOLS/MySQL/MySQL_Platform/Online_Schema_Changes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To safely drop a column from a large MySQL table, we need to ensure that no queries are reading or relying on this column. The steps involve checking the schema and query dependencies, running a non-destructive dry run, monitoring database performance, and finally executing the actual column removal.

#### Triage

The core of this question is about safely removing a potentially unused column from a 200 million-row table in MySQL. The interviewer is probing your understanding of:

Schema management.
Query dependency analysis.

#### Performance considerations
during large-scale changes.
Monitoring and validation techniques.

We assume the column's existence in the database schema, no immediate application logic that depends on it, and a general awareness of performance implications.

Step-by-Step Solution
1. Schema Analysis
Identify the Column: First, determine which column needs to be dropped.
Check Foreign Keys: Use SHOW CREATE TABLE or \d+ table_name in MySQL to check if any foreign keys reference this column.
2. Dependency Check
Query Dependency Analysis: Run a query like:
SQL
SELECT DISTINCT table_name, column_name
FROM information_schema.key_column_usage
WHERE referenced_table_name = 'table_name';
Application Logic: Verify if the column is used in any stored procedures or triggers using SHOW CREATE PROCEDURE/TRIGGER commands.
3. Non-Destructive Dry Run
Create a Backup: Take a full backup of the database and/or specific tables involved.
Modify Schema Temporarily: Use an ALTER TABLE statement to add a soft delete flag or mark the column as unused:
SQL
ALTER TABLE table_name MODIFY column_name datatype NOT NULL DEFAULT 'unused';
Monitor Queries: Implement logging for queries accessing this column. Use EXPLAIN and trace logs in MySQL to monitor usage.
4. Performance Monitoring
Baseline Performance: Run performance benchmarks before making any changes.
Post-Drop Analysis: After the dry run, analyze the impact on query performance and system load.
5. Execute Destructive Migration
Confirm No Dependencies: Re-check that no queries are using the column.
Drop Column Safely:
SQL
ALTER TABLE table_name DROP COLUMN column_name;
Validate Change: Ensure there are no lingering references or queries.
6. Post-Migration Verification
Re-run Baseline Checks: Confirm that the application and database behave as expected.
Documentation: Document the change, its rationale, and any follow-up actions.

#### Closing Summary

To safely drop a column from a large table in MySQL, you must first ensure no dependencies exist. This involves schema analysis, dependency checks, non-destructive dry runs, performance monitoring, executing the change carefully, and validating post-migration. Common pitfalls include overlooking dependent queries or not thoroughly testing changes.

#### Common Mistakes/Follow-Ups
Failing to Check Dependencies: Not using tools like information_schema can lead to missing critical dependencies.
Ignoring Performance Impact: Failing to monitor and measure the performance during dry runs and after migration can result in unexpected downtime or degraded service.
Lack of Backup: Missing out on taking a proper backup can result in data loss if something goes wrong.

