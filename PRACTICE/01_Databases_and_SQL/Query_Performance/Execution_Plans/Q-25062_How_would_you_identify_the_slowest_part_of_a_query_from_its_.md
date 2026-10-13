---
type: "practice"
difficulty: "Intermediate"
---

# How would you identify the slowest part of a query from its EXPLAIN ANALYZE output?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Diagnosing Query Regressions](<../../../../01_Databases_and_SQL/Query_Performance/Execution_Plans/Diagnosing_Query_Regressions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To identify the slowest part of a query from its EXPLAIN ANALYZE output, it's crucial to understand that this tool provides detailed insights into how PostgreSQL (or any SQL database) executes your queries. By analyzing the output, you can pinpoint which parts of the query are causing performance bottlenecks. Here’s a step-by-step guide:

Firstly, remember that EXPLAIN ANALYZE breaks down the execution plan and times each operation, allowing you to see not just how the query will be executed but also how long each part takes. This can help you identify which parts of your query are consuming the most resources.

**Code:**

```sql
-- Example EXPLAIN ANALYZE output (simplified for illustration)
EXPLAIN ANALYZE SELECT * FROM customers WHERE city = 'New York' AND age > 30;
```

**Example input / output:**

```

Here’s an example of what the EXPLAIN ANALYZE output might look like:

CODE
Copy
QUERY PLAN
-------------------------------------------------------------------------------------------------------------------------
 Index Scan using idx_customers_city ON customers  (cost=10.25..604.37 rows=98 width=32) (actual time=0.043..0.562 rows=16 loops=1)
   Filter: ((city)::text = 'New York'::text AND age > 30)
   Rows Removed by Filter: 1
 Planning Time: 0.098 ms
 Execution Time: 0.754 ms
```

