---
type: "practice"
difficulty: "Deep"
---

# A view is used in a hot path and you see repeated recompilation; explain how view definition changes invalidate cached plans and how to minimize churn.

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Materialized View Refresh Correctness](<../../../../01_Databases_and_SQL/Database_Objects/Views/Materialized_View_Refresh_Correctness.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of repeated recompilation of views in a hot path, we need to understand why views can lead to plan invalidation and explore strategies to minimize this churn. Views are essentially cached materialized results that improve query performance by reducing the complexity of queries that repeatedly hit the same data subset. However, changes in view definitions can invalidate the cached execution plans, leading to recompilation.

TL;DR

Changes in a view's definition can invalidate cached execution plans, necessitating repeated recompilations in hot paths. To minimize churn, consider using materialized views with refresh policies and carefully manage schema changes.

#### Triaging the Question

The core of this question lies in understanding why view definitions are invalidated and how to mitigate these issues. The interviewer is probing your knowledge of query optimization, caching strategies, and database management techniques. Key assumptions include:

The current implementation uses a hot path that frequently accesses the view.
There have been changes in the view definition that trigger recompilation.
Step-by-Step Solution

Understanding Plan Invalidations:

SQL databases use query execution plans to optimize queries. These plans are cached based on the schema and data of the underlying tables.
Changes to a view's definition, such as altering its structure or the source tables it references, can invalidate these cached plans.

Why Views Need Recompilation:

When a view is created, an execution plan is generated based on the current state of the underlying data and schema.
Any change in the view definition—such as adding columns, modifying the query logic, or changing the base tables—forces the database to re-examine the view's plan.

Minimizing Churn:

Use Materialized Views: Instead of using standard views, consider materialized views that store the results in a persistent table. This reduces the need for real-time computation and minimizes recompilation.
Refresh Policies: Configure refresh policies to control when materialized views are updated. For instance, you can set up periodic refreshes during off-peak hours or use change data capture (CDC) to update views based on changes in the underlying tables.
Schema Change Management:
Implement versioning for schema changes to avoid breaking existing plans. Incremental updates and staged deployments can help manage these transitions smoothly.
Use database-specific features like DDL triggers that allow you to intercept schema changes before they take effect, allowing time to optimize the view accordingly.

Best Practices:

Regularly review and update materialized views as part of your database maintenance strategy.
Monitor query performance and recompile plans only when necessary, balancing between recompilation costs and execution plan freshness.

#### Closing Summary

In summary, repeated recompilation of views in hot paths can be mitigated by using materialized views with appropriate refresh policies. Careful management of schema changes and periodic reviews are essential to maintain optimal query performance without excessive churn. Common mistakes include over-relying on standard views for complex queries and failing to implement robust refresh strategies.

#### Common Mistakes
Overlooking Refresh Policies: Not configuring or properly managing how materialized views get updated.
Ignoring Schema Changes: Failing to adapt view definitions when the underlying schema changes, leading to unnecessary recompilations.
Underestimating Maintenance: Ignoring regular maintenance and optimization of both standard and materialized views.

**Code:**

```sql
-- Example of creating a materialized view with a refresh policy
CREATE MATERIALIZED VIEW sales_summary AS
SELECT customer_id, SUM(sales_amount) as total_sales
FROM sales_data
GROUP BY customer_id;

-- Refresh the materialized view during off-peak hours
REFRESH MATERIALIZED VIEW sales_summary AT TIME '23:00' ON SCHEDULE EVERY 1 DAY;
```

**Flow:**

1. Query Execution
2. Read from Materialized View
3. Change in Base Table
4. Recompute Plan for View
5. Refresh Materialized View
6. Reduce Recompilation Churn
7. Schema Change
8. Adjust View Definition

