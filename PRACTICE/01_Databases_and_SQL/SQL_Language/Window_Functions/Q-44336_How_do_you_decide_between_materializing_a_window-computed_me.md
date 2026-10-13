---
type: "practice"
difficulty: "Deep"
---

# How do you decide between materializing a window-computed metric into a summary table versus recomputing it on each query for a metric read thousands of times per minute?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Window Function Fundamentals and Performance](<../../../../01_Databases_and_SQL/SQL_Language/Window_Functions/Window_Function_Fundamentals_and_Performance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deciding whether to materialize window-computed metrics into a summary table or recompute them on each query, we need to balance between performance and storage costs. Let's break this down from first principles.

First, window functions are used in SQL to compute results over a set of rows that are related to the current row in some way, often for time-based aggregations like running totals or moving averages. Materializing these metrics into a summary table involves pre-computing them and storing the results, which can be beneficial when querying the same windowed data frequently.

Now, let's triage the key aspects of this decision:

Query Frequency: If the metric is read thousands of times per minute, materialization could significantly reduce query latency.
Data Volume and Computation Complexity: More complex or voluminous data may benefit more from materialization due to increased computation cost on each query.
Storage Costs vs. Query Performance: Materialized views can be expensive in terms of storage but are cheap in terms of read performance.

To reason through this step by step:

#### Assumptions

The metric is window-based, such as a moving average or rolling sum.
The data volume and computation complexity are significant.
Storage costs for materialized views are high due to the need for storing intermediate results.

Computation Complexity: If the computation involves simple aggregation (e.g., SUM, AVG) on large datasets, it might be more efficient to recompute directly from raw data. However, if the computation is complex or time-series-based (e.g., moving averages), materialization could provide substantial performance benefits.

Query Performance:

Materialized views are faster for read-heavy workloads as they avoid expensive computations.
For write-heavy systems, the overhead of maintaining a materialized view must be considered, especially if the underlying data changes frequently.

Storage and Cost: Materializing may reduce query costs but increase storage costs. We need to ensure that the cost savings from reduced query times outweigh the additional storage expenses.

Maintenance Overhead:

Updating materialized views requires periodic refreshes, which can be costly in terms of processing time.
If the underlying data changes frequently, maintaining a fresh and accurate view is challenging.

#### Trade-offs
For metrics that are read thousands of times per minute, materialization could save significant query time. However, for infrequently accessed metrics or simpler computations, recomputation might be more efficient.

Given these considerations, we can outline the steps to decide:

Estimate Query Latency:

If the current query latency is high due to complex computations, materializing could significantly reduce it.

Assess Storage Costs:

Evaluate the cost of storing the intermediate results versus the cost of maintaining fresh computed values.

Implement a Proof-of-Concept:

Create both scenarios (materialized view and recomputation) with sample data to measure actual performance gains/losses.

Monitor Performance:

Continuously monitor query performance and storage costs post-deployment.

Here’s a simplified example using SQL:

SQL
-- Example of a window function in raw form
SELECT id, value,
       SUM(value) OVER (ORDER BY time ROWS BETWEEN 1 PRECEDING AND CURRENT ROW) as running_sum
FROM data_table;

-- Materialized view approach
CREATE MATERIALIZED VIEW mv_running_sum AS
SELECT id, value,
       SUM(value) OVER (ORDER BY time ROWS BETWEEN 1 PRECEDING AND CURRENT ROW) as running_sum
FROM data_table;

In practice, you would use a tool like Apache Calcite or an equivalent to optimize and materialize the views based on query patterns.

#### Closing Summary
Deciding between materializing window-computed metrics into summary tables versus recomputing them each time involves evaluating query frequency, computation complexity, storage costs, and maintenance overhead. A thorough analysis through proof-of-concept and continuous monitoring will provide insights for making an informed decision.

#### Common Mistakes/Follow-Ups

Overlooking the maintenance cost of materialized views.
Assuming all window functions are too complex to recompute, which might not be true for simple aggregations.
Not considering the frequency of data updates when deciding on freshness requirements.

**Comparison:**

```
FLOWCHART · MERMAID
gantt
    title Materialized View vs Recomputation for Window Functions
    section Analysis
        Window Function Computation: t1, 0d
        Query Performance Measurement: t2, 1d
        Storage Cost Assessment: t3, 1d
        Maintenance Overhead Estimation: t4, 1d
    section Implementation
        Create Materialized View: t5, 2d
        Deploy and Monitor: t6, 5d
    section Monitoring
        Performance Tuning: t7, 2d
```

**Code:**

```sql
-- Example SQL code for raw computation and materialization
SELECT id, value,
       SUM(value) OVER (ORDER BY time ROWS BETWEEN 1 PRECEDING AND CURRENT ROW) as running_sum
FROM data_table;

CREATE MATERIALIZED VIEW mv_running_sum AS
SELECT id, value,
       SUM(value) OVER (ORDER BY time ROWS BETWEEN 1 PRECEDING AND CURRENT ROW) as running_sum
FROM data_table;
```

**Example input / output:**

```text
Input:
SELECT id, value,
       SUM(value) OVER (ORDER BY time ROWS BETWEEN 1 PRECEDING AND CURRENT ROW) as running_sum
FROM data_table;

Output:
id | value | running_sum
----------------------
1  | 10    | NULL
2  | 20    | 30
3  | 30    | 60

Input (Materialized View):
SELECT * FROM mv_running_sum;

Output:
id | value | running_sum
----------------------
1  | 10    | NULL
2  | 20    | 30
3  | 30    | 60
```

**Data:**

```json
{
  "data_table": [
    {"id": 1, "value": 10, "time": "2023-01-01"},
    {"id": 2, "value": 20, "time": "2023-01-02"},
    {"id": 3, "value": 30, "time": "2023-01-03"}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Query Frequency
4. Materialize?
5. Maintain Materialized View
6. Recompute on Each Query
7. Maintain Freshness
8. Compute On-the-Fly

