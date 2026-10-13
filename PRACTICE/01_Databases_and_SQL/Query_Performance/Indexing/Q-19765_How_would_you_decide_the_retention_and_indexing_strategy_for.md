---
type: "practice"
difficulty: "Deep"
---

# How would you decide the retention and indexing strategy for pg_stat_statements or the slow query log itself, so your observability doesn't become a performance problem?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Index-only Scans](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Index-only_Scans.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To tackle the challenge of deciding the retention and indexing strategy for pg_stat_statements in PostgreSQL, we need to balance observability with performance concerns. This involves understanding the nature of slow queries, assessing their frequency, and determining how much historical data is useful without overwhelming the system.

#### Triaging the Question

The core of this question revolves around optimizing the database by setting appropriate retention policies for pg_stat_statements, which captures information about executed queries. The interviewer is probing your ability to:

Understand query performance metrics.
Evaluate trade-offs between data retention and storage costs.
Design efficient indexing strategies.

#### Cons
ider system scalability and resource management.

#### First Principles Solution

Assess Query Frequency and Impact:

Identify the most common slow queries that impact application performance.
Determine if these queries are critical enough to warrant detailed analysis over time.

Set Retention Policies:

For frequent, high-impact queries, consider a shorter retention period (e.g., 30 days) to quickly identify trends and anomalies.
For less impactful or infrequent queries, a longer retention period (e.g., 90 days) can be more cost-effective.

Indexing Strategy:

Index pg_stat_statements based on query patterns and frequency.
Use b-tree indexes for frequent searches by user_id, query text, or response time to speed up data retrieval.

Monitor Storage Costs and Performance:

Regularly review the size of pg_stat_statements tables to ensure they do not grow uncontrollably.
Optimize queries that are frequently executed but result in high storage costs.

Implement Automated Policies:

Use PostgreSQL's built-in tools like pg_stat_statements along with custom scripts or third-party monitoring tools to automate retention and optimization decisions.
Detailed Steps

Analyze Slow Queries:

SQL
SELECT query, calls, total_time
FROM pg_stat_statements
ORDER BY total_time DESC LIMIT 50;

This will give you a list of the most time-consuming queries. Analyze these to understand their impact on application performance.

Set Up Retention Policies:

Use pg_stat_statements's built-in feature for retention policies.
Configure PostgreSQL to archive and delete old data:
SQL
ALTER DATABASE your_database_name SET pg_stat_statements.max = 10000;
ALTER DATABASE your_database_name SET pg_stat_statements.track = ALL;

Indexing:

Create indexes on frequently queried columns:
SQL
CREATE INDEX idx_query_on_user_id ON pg_stat_statements (user_id);
CREATE INDEX idx_query_text ON pg_stat_statements USING btree(query);
Monitor index performance and adjust as needed.

Monitor Storage Costs:

Use pg_statistic to monitor the growth of pg_stat_statements.
Implement scripts or tools to automatically shrink large tables:
SQL
VACUUM FULL pg_stat_statements;

Automate and Optimize:

Develop automated scripts to manage retention policies based on query impact.
Use alerts and notifications to alert DBAs when storage costs exceed thresholds.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, the key is to balance observability with performance by setting appropriate retention policies and optimizing indexing strategies. Overly aggressive retention can lead to high storage costs and slower query performance, while insufficient data might miss critical insights into application behavior.

#### Common mistakes

Ignoring frequent yet low-impact queries.
Not regularly reviewing and adjusting retention policies.
Failing to monitor index effectiveness and storage growth.

#### Follow-Ups
How would you handle a scenario where the database needs to be downsized due to performance issues?
What other tools or metrics would you consider for optimizing query performance?

**Comparison:**

```
FLOWCHART · MERMAID
gantt
    title pg_stat_statements Retention and Indexing Strategy
    section Analyze Queries
        q_1: Analyze the top 50 most time-consuming queries using `pg_stat_statements` - d
    section Set Retention Policies
        r_1: Configure retention policies for frequent, high-impact queries - d+2w
    section Indexing Strategy
        i_1: Create indexes on frequently queried columns - d+3w
    section Monitor Storage Costs
        s_1: Regularly review the size of `pg_stat_statements` tables - mthly
    section Automate Policies
        a_1: Develop automated scripts to manage retention policies based on query impact - qtrly
```

**Code:**

```sql
-- Example of setting retention policies
ALTER DATABASE your_database_name SET pg_stat_statements.max = 10000;
ALTER DATABASE your_database_name SET pg_stat_statements.track = ALL;

-- Creating indexes
CREATE INDEX idx_query_on_user_id ON pg_stat_statements (user_id);
CREATE INDEX idx_query_text ON pg_stat_statements USING btree(query);

-- Shrinking large tables
VACUUM FULL pg_stat_statements;
```

**Example input / output:**

```

Input:

Analyze the top 50 most time-consuming queries using pg_stat_statements.

Output:

Queries: SELECT * FROM products WHERE price > 1000 ORDER BY created_at DESC, SELECT * FROM orders WHERE user_id = 123456.
Calls and total_time for these queries.

Dry Run: Input:

SQL
Copy
SELECT query, calls, total_time
FROM pg_stat_statements
ORDER BY total_time DESC LIMIT 50;

Output:

JSON
Copy
[
  {"query": "SELECT * FROM products WHERE price > 1000 ORDER BY created_at DESC", "calls": 50, "total_time": 20.5},
  {"query": "SELECT * FROM orders WHERE user_id = 123456", "calls": 80, "total_time": 12.0}
]
```

**Data:**

```json
{
  "queries": [
    {
      "query": "SELECT * FROM products WHERE price > 1000 ORDER BY created_at DESC",
      "calls": 50,
      "total_time": 20.5
    },
    {
      "query": "SELECT * FROM orders WHERE user_id = 123456",
      "calls": 80,
      "total_time": 12.0
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Analyze top 50 queries
5. Frequent & High-Impact?
6. Set short retention policy
7. Set longer retention policy
8. Create indexes on criticalcolumns
9. Monitor storage costs
10. Automate policies based onimpact
11. Review and adjust
12. Implement automated scripts
13. End

