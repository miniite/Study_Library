---
type: "practice"
difficulty: "Expert"
---

# How do you enable and verify that query rewrite is actually happening for a given SQL statement?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Materialized View Refresh and Maintenance](<../../../../01_Databases_and_SQL/Database_Objects/Materialized_Views/Materialized_View_Refresh_and_Maintenance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: To enable and verify query rewrites with materialized views, we need to set up the materialized view, ensure it's properly configured, and then confirm that queries are being rewritten by comparing execution plans or monitoring query performance.

When working with materialized views (MV) in a database environment, one of the key benefits is that certain SQL statements can be rewritten to leverage existing MVs. This optimization is particularly useful for read-heavy operations where precomputed data reduces the load on the underlying tables. To ensure this feature works as expected, we need to follow several steps: setting up the materialized view, enabling rewrite functionality, and verifying that queries are indeed being rewritten.

#### Triage

The question is asking how to enable and verify query rewrites with a materialized view (MV). This involves understanding the setup process, confirming configuration settings, and validating whether the database system is correctly rewriting SQL statements using MVs. Key assumptions include:

The target database supports materialized views and query rewrites.
We have access to the necessary tools for managing and querying the database.

#### First Principles Solution

Setup Materialized View:

Define a materialized view that captures the desired data subset or aggregation.
Ensure the MV is refreshable and properly configured with appropriate update frequency or manual trigger points.

Enable Query Rewrite:

Configure the database to enable query rewrites for the materialized view.
Check if the rewrite is enabled via system parameters or settings specific to your database management system (DBMS).

Verify Rewrites:

Use EXPLAIN plans to analyze the execution plan of a given SQL statement before and after refresh.
Monitor query performance metrics to ensure there's an improvement in response time.
Step-by-Step Solution

Create Materialized View:

SQL
CREATE MATERIALIZED VIEW mv_sales_summary AS
SELECT customer_id, SUM(sales_amount) as total_sales
FROM sales_data
GROUP BY customer_id;

Enable Query Rewrite (DBMS-specific):

For example, in Oracle, use the ALTER SYSTEM SET optimizer_enable_rewrite=TRUE;
In PostgreSQL, ensure rewrite_enabled is set to true.

Refresh Materialized View:

SQL
REFRESH MATERIALIZED VIEW mv_sales_summary;

Verify Rewrite Using EXPLAIN Plan:

Before refresh:
SQL
EXPLAIN SELECT customer_id, SUM(sales_amount) as total_sales FROM sales_data GROUP BY customer_id;
After refresh:
SQL
EXPLAIN SELECT customer_id, SUM(sales_amount) as total_sales FROM mv_sales_summary;

Monitor Query Performance:

Compare the response times of the original query and the rewritten query.
Use database metrics like execution time, I/O operations, and memory usage.

#### Closing Summary
+ Common Mistakes/Follow-Ups
Ensure that all steps are correctly implemented to avoid common mistakes such as missing refresh intervals or incorrect configuration settings.

#### Follow-ups
might include discussing potential trade-offs between MVs (e.g., storage overhead vs. query performance) and exploring alternative optimization strategies like partitioning or indexing.

**Comparison:**

| Query           | Before Refresh  | After Refresh   |
|-----------------|-----------------|----------------|
| Execution Plan  | Full table scan | Direct access  |
| Response Time   | High            | Low            |

**Code:**

```sql
-- Example SQL for creating a materialized view
CREATE MATERIALIZED VIEW mv_sales_summary AS
SELECT customer_id, SUM(sales_amount) as total_sales
FROM sales_data
GROUP BY customer_id;

-- Example SQL for refreshing the materialized view
REFRESH MATERIALIZED VIEW mv_sales_summary;
```

**Example input / output:**

```text
Before refresh:
EXPLAIN SELECT customer_id, SUM(sales_amount) as total_sales FROM sales_data GROUP BY customer_id;
-- Output shows full table scan

After refresh:
EXPLAIN SELECT customer_id, SUM(sales_amount) as total_sales FROM mv_sales_summary;
-- Output should show a direct access to the materialized view
```

**Data:**

```json
{
  "sales_data": [
    {"customer_id": 101, "sales_amount": 500},
    {"customer_id": 102, "sales_amount": 700},
    {"customer_id": 103, "sales_amount": 900}
  ]
}
```

**Flow:**

1. Create Materialized View
2. Enable Query Rewrite
3. Refresh Materialized View
4. Verify Using EXPLAIN Plan
5. Monitor Performance Metrics
6. Confirm Rewrites Happening

