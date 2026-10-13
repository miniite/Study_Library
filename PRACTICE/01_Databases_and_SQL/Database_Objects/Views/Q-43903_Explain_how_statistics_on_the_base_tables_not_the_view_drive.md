---
type: "practice"
difficulty: "Deep"
---

# Explain how statistics on the base tables (not the view) drive the plan for a view query, and what happens when those stats go stale on a hot table.

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Using Views Effectively](<../../../../01_Databases_and_SQL/Database_Objects/Views/Using_Views_Effectively.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand how statistics on base tables influence the execution plan for a view query, we need to delve into how database systems use these statistics to make cost-based decisions. Statistics are crucial because they provide information about data distribution and cardinality, helping the optimizer choose an efficient execution strategy.

In essence, when you run a query against a view, the database system must translate that query into operations on the underlying base tables. The quality of the execution plan depends heavily on accurate statistics—specifically, those gathered from the base tables involved in the view definition. If these statistics are outdated or missing, the query optimizer may generate suboptimal plans.

Now let's break down what happens when base table statistics go stale and explore how this affects a hot table (a frequently accessed one).

**Code:**

```sql
-- Example of a simple view definition
CREATE VIEW sales_summary AS
SELECT product_id, SUM(sales_amount) as total_sales
FROM sales_data
GROUP BY product_id;
```

**Example input / output:**

```

For instance, consider the sales_summary view defined above. The query optimizer needs to decide how to efficiently retrieve and aggregate data from the sales_data table.

When base table statistics are current:

SQL
Copy
-- Optimizer can make informed decisions based on accurate statistics
EXPLAIN SELECT * FROM sales_summary WHERE product_id = 123;

This might result in an efficient execution plan, possibly using an index on product_id.

However, if the statistics for the sales_data table go stale:

SQL
Copy
-- Optimizer may choose a less optimal plan due to outdated statistics
EXPLAIN SELECT * FROM sales_summary WHERE product_id = 123;

This could lead to a full table scan instead of using an index, resulting in slower performance.
```

**Flow:**

1. If stats are stale or missing
2. Stats are up-to-date
3. Query Against View
4. Translate Query into Base TableOperations
5. Fetch Statistics for Base Tables
6. Generate SuboptimalExecution Plan
7. Generate Optimized ExecutionPlan

