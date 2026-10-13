---
type: "practice"
difficulty: "Expert"
---

# You observe that autovacuum keeps falling behind on a high-churn table, causing bloat and slow scans. How do you tune per-table autovacuum settings, and when is a scheduled aggressive VACUUM or pg_repack the right call?

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ MVCC Bloat and Vacuum](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/MVCC_Bloat_and_Vacuum.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with a high-churn table where autovacuum falls behind, causing bloat and slow scans, you need to carefully tune per-table autovacuum settings. This involves understanding the underlying issues and making informed decisions about when to use VACUUM or pg_repack. Let's break down the steps from first principles.

#### Triage
Understanding the Problem

Autovacuum is designed to manage table bloat by periodically performing VACUUM operations, but it can struggle with high-churn tables. The main issues are:

High Insert/Update Rate: Frequent insertions and updates lead to more dead tuples (rows marked for deletion).
Bloat Accumulation: Dead tuples accumulate over time, leading to increased table size.
Slow Scans: Queries may need to scan through a larger number of rows due to the bloat.

#### Assumptions
The table has a significant churn rate with many inserts and updates.
Autovacuum is configured at the database level but may not be optimal for this specific table.
The goal is to reduce bloat and improve query performance without overloading the system.
Step-by-Step Solution
1. Review Current Autovacuum Settings

First, review the current autovacuum settings using SHOW commands:

SQL
SHOW autovacuum_vacuum_scale_factor;
SHOW autovacuum_analyze_scale_factor;
SHOW min_autovacuum_vacuum_cost;

Identify if these global settings are too conservative or aggressive for your specific table.

2. Tune Per-Table Autovacuum Settings

You can set per-table autovacuum settings using ALTER TABLE. For example:

SQL
ALTER TABLE high_churn_table SET (
    autovacuum_vacuum_scale_factor = 0.1,
    autovacuum_analyze_scale_factor = 0.1,
    min_autovacuum_vacuum_cost = -10000
);
autovacuum_vacuum_scale_factor: Adjusts the number of rows to scan before performing a VACUUM.
autovacuum_analyze_scale_factor: Controls when analyze is performed for statistics collection.
min_autovacuum_vacuum_cost: Sets the minimum cost threshold for autovacuum operations.
3. Monitor and Optimize

After setting the per-table settings, monitor the performance:

SQL
VACUUM VERBOSE HIGH_CHURN_TABLE;
EXPLAIN ANALYZE SELECT * FROM high_churn_table WHERE some_column = 'value';

Adjust the settings based on observed behavior and performance metrics.

4. Consider Scheduled Aggressive VACUUM or pg_repack

If tuning autovacuum does not suffice, consider scheduling an aggressive VACUUM FULL or using pg_repack. These can be resource-intensive but provide a one-time cleanup.

SQL
-- Schedule a VACUUM FULL
VACUUM FULL HIGH_CHURN_TABLE;

-- Use pg_repack for more granular control and lower downtime
SELECT * FROM pg_repack.repack('high_churn_table');

#### Closing Summary

By carefully tuning per-table autovacuum settings, you can address the issues of high-churn tables. However, if these do not resolve the problem, scheduled aggressive VACUUM or pg_repack might be necessary. Always monitor and adjust based on performance metrics to find the optimal balance.

#### Common Mistakes/Follow-Ups
Not Monitoring: Failing to continuously monitor performance can lead to suboptimal settings.
Overzealous Settings: Aggressive settings without proper monitoring can cause system overload.
Ignoring Global Settings: Not considering how global autovacuum settings affect your table.
Frequent VACUUM FULL: This is resource-intensive and should be used sparingly.

**Code:**

```sql
-- Example of setting per-table autovacuum settings
ALTER TABLE high_churn_table SET (
    autovacuum_vacuum_scale_factor = 0.1,
    autovacuum_analyze_scale_factor = 0.1,
    min_autovacuum_vacuum_cost = -10000
);
```

**Example input / output:**

```text
-- Example output of VACUUM and EXPLAIN ANALYZE
VACUUM VERBOSE HIGH_CHURN_TABLE;
EXPLAIN ANALYZE SELECT * FROM high_churn_table WHERE some_column = 'value';
```

