---
type: "practice"
difficulty: "Expert"
---

# Autovacuum can't keep up on a hot table and bloat is growing; how do you diagnose it with pg_stat_user_tables and tune per-table autovacuum without a maintenance window?

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ Finding Expensive Queries](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Finding_Expensive_Queries.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with a PostgreSQL database where autovacuum can't keep up, leading to growing bloat on a hot table, our primary goal is to diagnose the root cause and tune autovacuum settings without disrupting ongoing operations. This involves leveraging pg_stat_user_tables to gather detailed information about the table's activity and vacuum statistics.

Triaging the Problem

Question Breakdown:

What are we diagnosing? We need to determine why the autovacuum process is lagging behind, leading to increasing bloat.
What tools are available for diagnosis? pg_stat_user_tables provides critical insights into table activity and vacuum statistics.
What tuning options do we have? We can adjust autovacuum settings per-table to ensure it catches up with the workload.
Step-by-Step Solution

Gather Table Statistics:

Use pg_stat_user_tables to inspect the autovacuum_analyze_count, vacuum_count, and last_autovacuum fields for the problematic table.
SQL
SELECT
    schemaname,
    tablename,
    autovacuum_analyze_count,
    vacuum_count,
    last_autovacuum
FROM
    pg_stat_user_tables
WHERE
    tablename = 'your_table_name';

Analyze Vacuum Patterns:

Check the last_autovacuum timestamp to see if it has been a long time since the last vacuum.
Analyze the autovacuum_analyze_count and vacuum_count to understand the frequency of these operations.

Identify Bloat Causes:

Use pgstattuple to determine the bloat percentage in the table:
SQL
SELECT
    schemaname,
    tablename,
    relpages::bigint * 8 / current_setting('block_size') AS total_pages,
    (reltuples * 20 / 100) AS dead_tuples,
    ((reltuples * 20 / 100) + (relpages::bigint * 8 / current_setting('block_size'))) - relpages::bigint * 8 / current_setting('block_size') AS bloat_pages
FROM
    pg_stat_user_tables,
    pg_class
WHERE
    pg_class.relname = 'your_table_name' AND
    schemaname = 'public';

Adjust Autovacuum Settings:

If the autovacuum is significantly behind and bloat is growing, consider increasing autovacuum_vacuum_threshold and autovacuum_analyze_threshold.
SQL
ALTER TABLE your_table_name SET (autovacuum_vacuum_threshold = 1000);
ALTER TABLE your_table_name SET (autovacuum_analyze_threshold = 500);
If the table has a high number of dead tuples, you might need to set autovacuum_freeze_max_age appropriately:
SQL
ALTER TABLE your_table_name SET (autovacuum_freeze_max_age = 200000000);

Monitor and Tune:

Continuously monitor the table's vacuum statistics to ensure that adjustments are effective.
If bloat continues, consider increasing autovacuum frequency or adding a maintenance window.

#### Closing Summary

By leveraging pg_stat_user_tables and pgstattuple, we can diagnose why autovacuum is lagging and identify the root cause of table bloat. Adjusting autovacuum settings per-table without a maintenance window ensures that critical operations continue smoothly while addressing the underlying issue.

#### Common Mistakes/Follow-Ups
Overly Aggressive Settings: Ensuring that autovacuum thresholds are not set too high, leading to excessive resource usage.
Ignoring Maintenance Windows: Failing to schedule periodic maintenance tasks can lead to compounding bloat issues.
Insufficient Monitoring: Not continuously monitoring the table's vacuum statistics can result in ongoing issues.

**Code:**

```sql
-- Example SQL queries for diagnosing and tuning autovacuum
SELECT
    schemaname,
    tablename,
    autovacuum_analyze_count,
    vacuum_count,
    last_autovacuum
FROM
    pg_stat_user_tables
WHERE
    tablename = 'your_table_name';

SELECT
    schemaname,
    tablename,
    relpages::bigint * 8 / current_setting('block_size') AS total_pages,
    (reltuples * 20 / 100) AS dead_tuples,
    ((reltuples * 20 / 100) + (relpages::bigint * 8 / current_setting('block_size'))) - relpages::bigint * 8 / current_setting('block_size') AS bloat_pages
FROM
    pg_stat_user_tables,
    pg_class
WHERE
    pg_class.relname = 'your_table_name' AND
    schemaname = 'public';

-- Example SQL for tuning autovacuum settings
ALTER TABLE your_table_name SET (autovacuum_vacuum_threshold = 1000);
ALTER TABLE your_table_name SET (autovacuum_analyze_threshold = 500);
ALTER TABLE your_table_name SET (autovacuum_freeze_max_age = 200000000);
```

