---
type: "practice"
difficulty: "Deep"
---

# How do you plan the ANALYZE / statistics-refresh step of a big migration so the optimizer doesn't chase stale stats on the newly-shaped table right after cutover?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Migrating to Partitioned Tables](<../../../../01_Databases_and_SQL/Database_Operations/Migrations/Concurrent_Index_Builds/Migrating_to_Partitioned_Tables.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that the query optimizer uses accurate statistics in a database post-migration, the key is to refresh the schema and statistics of the new tables while minimizing disruption during the migration process. This involves carefully planning the timing and scope of the ANALYZE step within your overall migration strategy.

The question here is how to ensure that the query optimizer has up-to-date information about the distribution of data in a newly structured or partitioned table, especially when you have a large-scale migration with many changes. The interviewer might be probing into understanding the impact of stale statistics on query performance, the importance of timing these operations correctly, and the broader implications for database management.

#### Triaging the Question
Understanding Stale Statistics: When tables are altered (e.g., through schema changes or data reorganization), the existing statistics can become outdated, leading to suboptimal query plans.
Post-Cutover Timing: Immediately after a migration, it's critical that you refresh the statistics on the new table structure before queries start running against it.
Scope of Refresh: You need to balance between refreshing enough tables to ensure good performance and not overloading the system with too many ANALYZE commands.

#### First-Principles Solution
Identify Affected Tables: Determine which tables have been significantly altered by the migration process.
Incremental Refresh Strategy: Instead of analyzing all affected tables at once, consider an incremental approach to ensure timely and efficient updates.
Schedule Analyzes: Plan these analyses during periods of low activity to minimize impact on user operations.
Use Dynamic Analysis Scheduling: Leverage database features that allow for dynamic scheduling of analyze tasks based on usage patterns.
Step-by-Step Solution
Review Migration Changes:
Identify tables with significant schema changes (e.g., column additions, deletions, or renamings).
Determine if the data distribution has changed significantly due to data reorganization.
Prioritize Analyzes:
Focus on critical queries that will run against these newly structured tables post-migration.
Schedule Analyzes Incrementally:
For large migrations, consider analyzing one or a few key tables at a time.
Use dynamic scheduling tools (if available) to analyze the most used queries first.
Monitor and Adjust:
Continuously monitor query performance post-migration and adjust the analysis schedule as needed.

#### Closing Summary

By carefully planning and executing an incremental strategy for analyzing newly structured tables, you can ensure that the optimizer uses accurate statistics without causing excessive system load or downtime during a migration.

#### Common Mistakes/Follow-Ups
Over-analyzing: Analyzing too many tables at once can overwhelm the database and impact performance.
Underestimating Impact: Failing to consider how outdated statistics affect query plans, leading to suboptimal execution times.
Ignoring Usage Patterns: Not scheduling analyzes during off-peak hours or based on actual usage patterns.

#### Conclusion

Proper planning of the ANALYZE step is crucial for maintaining optimal database performance post-migration. By adopting an incremental and well-timed approach, you can minimize disruption and ensure that your queries run efficiently.

**Comparison:**

```
Yes
No
Identify Affected Tables
Are there significant schemachanges?
Prioritize Analyzes
Analyze Key Tables
No Analysis Needed
```

**Code:**

```sql
-- Example of an incremental analyze strategy

-- Step 1: Identify affected tables
SELECT table_name
FROM information_schema.tables
WHERE table_schema = 'your_schema' AND table_name IN ('table1', 'table2');

-- Step 2: Analyze specific tables incrementally
ANALYZE TABLE table1;
ANALYZE TABLE table2;

-- Step 3: Schedule dynamic analysis (if supported)
SELECT * FROM pg_stat_statements
ORDER BY total_exec_time DESC LIMIT 5; -- Identify top queries and schedule analyzes accordingly
```

**Example input / output:**

```

Input:

Identify and prioritize affected tables. Output:
Incrementally analyze key tables to ensure accurate statistics.

Dry-run:

Query identified tables:

CODE
Copy
SELECT table_name
FROM information_schema.tables
WHERE table_schema = 'your_schema' AND table_name IN ('table1', 'table2');

Analyze selected tables:

CODE
Copy
ANALYZE TABLE table1;
ANALYZE TABLE table2;
```

**Data:**

```json
{
  "affected_tables": [
    { "name": "table1", "schema": "your_schema" },
    { "name": "table2", "schema": "your_schema" }
  ]
}
```

**Flow:**

1. Start Migration
2. Identify Affected Tables
3. Analyze Key TablesIncrementally
4. Schedule Dynamic Analysis
5. Monitor and Adjust
6. Post-Migration Optimization

