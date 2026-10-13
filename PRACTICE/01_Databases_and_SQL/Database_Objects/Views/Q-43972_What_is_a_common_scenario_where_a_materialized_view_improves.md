---
type: "practice"
difficulty: "Intermediate"
---

# What is a common scenario where a materialized view improves reporting or dashboard performance?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Materialized View Trade-offs](<../../../../01_Databases_and_SQL/Database_Objects/Views/Materialized_View_Trade-offs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Materialized views are a powerful tool in database development, especially when dealing with complex queries that are used frequently for reporting and dashboards. A common scenario where materialized views significantly improve performance is when you have a large dataset combined with complex SQL queries that are run repeatedly.

#### Triaging the Question

The core of this question revolves around understanding the benefits and use cases of materialized views (MVs). The interviewer might be probing your knowledge on database optimization techniques, specifically in scenarios where read-heavy operations are common. Assumptions include familiarity with query performance issues and the trade-offs between storage and real-time data freshness.

#### First-Principles Solution

Materialized views store the results of a precomputed query as a physical table. This can be extremely useful when you have complex queries that take a long time to execute, but are queried frequently. The key idea is to cache the result set at a point in time, which can then be retrieved much faster than recalculating the full query.

Scenario: Complex Reporting Queries

Imagine a scenario where your business needs regular reporting on various metrics such as revenue by product category and geographical breakdown over multiple years. Without materialized views, running these reports might involve complex queries with JOINs, GROUP BY clauses, and subqueries that take several seconds to complete. However, if you create a materialized view for this report, the results are stored in a physical table.

When the reporting query is executed, it simply retrieves data from the materialized view instead of recalculating everything each time. This can significantly reduce query execution times, making the reports available much faster and improving user experience.

#### Trade-offs

While materialized views offer performance benefits, they come with trade-offs:

Storage: Materialized views require additional storage space to store the precomputed results.
Data Freshness: If the underlying data changes frequently, you need a strategy to keep the materialized view up-to-date without compromising its performance.

#### Closing Summary

In summary, materialized views are particularly beneficial in scenarios where complex queries run repeatedly and slow down reporting or dashboard updates. They can significantly enhance query performance by caching results, but careful consideration of storage requirements and data freshness is necessary.

#### Common Mistakes/Follow-ups
Forgetting to regularly refresh the materialized view when underlying data changes.
Overusing materialized views for simple queries where the overhead outweighs the benefits.
Not understanding that materialized views are not real-time; they store a snapshot of the query results at a certain point in time.

**Comparison:**

| Product Category | Total Revenue |
|------------------|---------------|
| Electronics      | 110,000       |
| Clothing         | 30,000        |

**Code:**

```sql
-- Example of creating a materialized view

CREATE MATERIALIZED VIEW sales_summary AS
SELECT product_category, SUM(revenue) as total_revenue
FROM sales_data
WHERE date BETWEEN '2021-01-01' AND CURRENT_DATE
GROUP BY product_category;
```

**Example input / output:**

```text
-- Query execution without materialized view
EXPLAIN ANALYZE SELECT product_category, SUM(revenue) as total_revenue FROM sales_data WHERE date BETWEEN '2021-01-01' AND CURRENT_DATE GROUP BY product_category;

-- Query execution with materialized view
EXPLAIN ANALYZE SELECT * FROM sales_summary;
```

**Data:**

```json
{
  "sales_data": [
    {"product_category": "Electronics", "revenue": 50000, "date": "2021-04-01"},
    {"product_category": "Clothing", "revenue": 30000, "date": "2021-05-01"},
    {"product_category": "Electronics", "revenue": 60000, "date": "2021-06-01"}
  ]
}
```

**Flow:**

1. Slow Response
2. Complex Query
3. Materialized View
4. Cache Results
5. Query Execution
6. Fast Response
7. Without Materialized View
8. Complex Computation
9. Slower Response

