---
type: "practice"
difficulty: "Expert"
---

# Autovacuum keeps falling behind on a high-churn table. How would you tune per-table autovacuum settings, and when is pg_repack or a scheduled aggressive VACUUM the right call?

**Difficulty:** Expert · **Levels:** SDE-1, SDE-1/2 · [↑ MVCC Bloat and Vacuum](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/MVCC_Bloat_and_Vacuum.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with a high-churn table where autovacuum keeps falling behind, the key is to fine-tune per-table autovacuum settings and possibly consider alternative approaches like pg_repack or scheduled aggressive VACUUM. The core idea here is to ensure that dead tuples are cleaned up promptly without overwhelming your system resources.

To effectively address this issue, we need to understand the underlying causes, such as high write throughput, frequent updates/deletes, or large transactions. The first step is to diagnose the problem by checking autovacuum logs and identifying tables with excessive bloat. Once identified, tuning specific parameters can help manage the situation more efficiently.

Triaging the Problem

Autovacuum falls behind when it cannot keep up with the rate of insertions, updates, or deletions in a table. This can lead to performance issues due to increased storage and query execution time. The primary concern is ensuring that dead tuples are vacuumed promptly to maintain database integrity and performance.

#### Assumptions
The table has a high churn rate (frequent inserts/updates/deletes).
Autovacuum settings may need adjustment.
There might be other factors contributing to the issue, such as concurrent workload or resource constraints.

#### First-Principles Solution

The first step is to tune autovacuum parameters for the specific table:

Increase autovacuum_vacuum_scale_factor: This parameter controls how many rows must change in a segment before a vacuum is initiated. Increasing it can help catch up with high churn, but be cautious as this could potentially delay necessary cleanups.

Adjust autovacuum_freeze_max_age and min_freeze_age: These settings control when data becomes frozen and needs to be moved to the new transaction ID (XID) table. Setting these values appropriately can prevent excessive XID bloat, but too low a value could lead to unnecessary vacuum activity.

Optimize autovacuum_vacuum_cost_delay and autovacuum_max_workers: These settings control how often autovacuum starts and the number of concurrent workers. Adjusting these can balance resource usage with cleanup needs.

#### Cons
ider Using pg_repack for large tables: If the table is very large, using pg_repack might be more effective than relying on autovacuum. This tool can rebuild the table, freeing up space and reducing bloat without downtime.

When to Use pg_repack or Aggressive VACUUM

Use pg_repack when:

The table is very large (e.g., gigabytes in size).
You need to free up significant amounts of space.
The table has a high churn rate and autovacuum settings are not sufficient.

Use Aggressive VACUUM when:

Autovacuum settings cannot be adjusted due to constraints (e.g., production environment with strict SLAs).
You need immediate cleanup without the overhead of pg_repack.
The table has a critical performance issue that requires urgent attention.

#### Closing Summary

In summary, tuning autovacuum parameters is crucial for managing high-churn tables. However, in cases where these adjustments are insufficient, tools like pg_repack or aggressive VACUUM can be employed to address the root cause of bloat and maintain database performance.

#### Common Mistakes
and Follow-Ups
Mistake 1: Overadjusting autovacuum settings without proper understanding. Always test changes in a staging environment.
Mistake 2: Ignoring concurrent workload or resource constraints when tuning parameters.

#### Follow-Up
Question: How do you monitor and verify the effectiveness of your tuning efforts?

**Code:**

```

-- Example of increasing autovacuum_vacuum_scale_factor ALTER TABLE high_churn_table SET (autovacuum_vacuum_scale_factor = 0.2);

CODE
Copy

```sql
-- Example of using pg_repack for large tables
SELECT * FROM pg_repack.repack('high_churn_table');
```

