---
type: "practice"
difficulty: "Deep"
---

# Explain when a BigQuery materialized view will and will not be used to accelerate a query automatically, and the limitations that surprise engineers.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ BigQuery Loading and Table Design](<../../../../TOOLS/Google_Cloud/BigQuery/BigQuery_Loading_and_Table_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When discussing BigQuery materialized views (Materialized Views or MVs), it's crucial to understand their role in accelerating queries. Materialized Views in BigQuery are precomputed datasets that store results of a query, allowing subsequent queries on the same data to be answered faster by querying these precomputed results rather than re-executing the underlying complex query.

MVs will be used automatically when:

Complex Queries: When a query is complex and executed frequently, an MV can significantly reduce execution time.
Join Operations: When joining large datasets, especially with smaller, more frequent tables, an MV on one of the larger tables can accelerate the join operation.
Filtering Operations: When filtering operations are common on specific columns, creating a materialized view that includes these filters can improve performance.

However, there are several scenarios where BigQuery will not use a Materialized View:

Recent Data: If data in the underlying tables is frequently updated and the MV hasn't refreshed yet, the MV might be outdated, leading to slower queries or incorrect results.
Dynamic Queries: When queries include dynamic elements such as parameters that change with each execution, an MV can’t be used because it’s based on a fixed query result.
Key Assumptions
The query's complexity and frequency are high enough to justify the overhead of maintaining the MV.
The underlying data is relatively static or changes infrequently compared to the use cases.
The query patterns are predictable, making it feasible to create an appropriate MV schema.
Step-by-Step First Principles Solution
Identify Query Patterns: Determine which queries are executed frequently and have complex operations like joins, filters, etc.
Analyze Data Change Frequency: Evaluate how often the underlying data changes compared to query execution frequency.
Design the MV Schema: Ensure that the MV schema accurately represents the most common and computationally intensive parts of the query.
Automate Refresh Policies: Configure refresh policies based on data change patterns to minimize staleness.
Limitations That Surprise Engineers
Staleness: Engineers often underestimate how quickly data might change, leading to a situation where MVs become outdated.
Overhead of Maintenance: The cost and complexity of maintaining an MV (including storage costs and refresh costs) can be higher than expected.
Query Complexity Trade-Offs: Simple queries may not benefit from an MV due to the overhead of creating and refreshing it, which can outweigh the benefits.

#### Closing Summary

Materialized Views in BigQuery are a powerful tool for accelerating query performance by caching results of complex queries. However, they require careful consideration of data change frequency, query complexity, and maintenance costs. Misunderstanding these factors can lead to suboptimal use or even negative performance impacts.

#### Common Mistakes
and Follow-Ups
Overly Complex MVs: Creating overly complex MVs that don’t align with the most common queries.
Ignoring Data Refresh Policies: Failing to configure appropriate refresh policies, leading to outdated data in MVs.
Neglecting Query Optimization: Assuming an MV will solve all performance issues without optimizing the underlying queries first.

**Comparison:**

| Data Change Frequency | MV Usage |
|-----------------------|----------|
| High (frequent updates)| Likely not used for accelerating queries, but may be used to store recent data. |
| Low (rare updates)     | More likely to benefit from an MV due to reduced refresh costs and staleness issues. |

**Code:**

```sql
-- Example of creating a materialized view
CREATE MATERIALIZED VIEW my_project.my_dataset.my_materialized_view AS
SELECT column1, SUM(column2) as total_sum
FROM my_project.my_dataset.original_table
GROUP BY column1;
```

