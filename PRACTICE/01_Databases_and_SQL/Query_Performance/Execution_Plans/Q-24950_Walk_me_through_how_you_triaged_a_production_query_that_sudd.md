---
type: "practice"
difficulty: "Intermediate"
---

# Walk me through how you triaged a production query that suddenly regressed from 50ms to 8 seconds without any code change.

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ Diagnosing Query Regressions](<../../../../01_Databases_and_SQL/Query_Performance/Execution_Plans/Diagnosing_Query_Regressions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview

When faced with a sudden regression in query performance, such as a query jumping from 50ms to 8 seconds without any code changes, the first step is to identify and analyze the execution plan of the query. Execution plans provide critical insights into how the database engine processes the query and can highlight bottlenecks or inefficiencies.

Triaging the Query Regression

The key question here is: How do we systematically diagnose the root cause of the performance issue by examining the execution plan? This involves understanding the structure of the execution plan, identifying any changes in the cost estimates or physical operations, and assessing whether there are new or altered indexes that might be affecting performance.

#### Assumptions
The database is a relational database (e.g., PostgreSQL, MySQL).
We have access to tools like EXPLAIN, EXPLAIN ANALYZE, or similar utilities provided by the database management system.
There are no code changes, so any regression must be due to execution plan variations.
Step-by-Step Solution

Capture the Execution Plan:

Use EXPLAIN (without ANALYZE) to get a high-level view of the query's execution strategy.
SQL
EXPLAIN SELECT * FROM some_table WHERE column = 'value';

This will give you an overview of the operations but won't include actual timing data.

Capture Detailed Execution Plan:

Use EXPLAIN ANALYZE to get detailed performance information.
SQL
EXPLAIN ANALYZE SELECT * FROM some_table WHERE column = 'value';

This will provide both the plan and runtime statistics, including row counts, costs, and actual elapsed times.

Compare Execution Plans:

Compare the execution plans before and after the performance degradation.
SQL
EXPLAIN (ANALYZE, VERBOSE) SELECT * FROM some_table WHERE column = 'value';

Look for any significant differences in operations, cost estimates, or actual runtimes.

Identify Key Differences:

Check if there are any changes in the query optimizer's choices.
New indexes may have been created accidentally.
Statistics on tables might be out of date, leading to suboptimal plans.
There could be changes in table data distribution affecting joins or scans.

Check for Indexes and Statistics:

Ensure that statistics are up-to-date using ANALYZE commands.
SQL
ANALYZE some_table;

Verify if any new indexes were added or existing ones dropped inadvertently, which could affect query plans significantly.

Analyze Table Data Distribution:

Check for data skew that might be causing disproportionate runtime increases.
Ensure that the distribution of data across partitions (if applicable) is optimal.

Review Query Cache and Plan Cache:

If caching mechanisms are involved, check if plan caching or query result caching is impacting performance.

Monitor Database Performance Metrics:

Use database-specific monitoring tools to see if there were any spikes in resource usage (CPU, memory, I/O) that could affect the query.
Check logs for any errors or warnings that might indicate issues with the database environment.

#### Closing Summary
and Common Mistakes

The key takeaway is that execution plans are dynamic and can change based on various factors. By systematically comparing the current plan with historical data, we can identify what has changed and address those changes to restore optimal performance. Common mistakes include ignoring statistics updates, overlooking new or dropped indexes, and failing to consider data skew.

Common Follow-Ups
How do you ensure that your queries are optimized for production?
What steps do you take when you find a significant change in the execution plan?
Have you ever encountered similar issues with other databases? How did you resolve them?

**Code:**

```sql
-- Example of Explain Analyze output
EXPLAIN ANALYZE SELECT * FROM some_table WHERE column = 'value';
```

