---
type: "practice"
difficulty: "Expert"
---

# Explain how a view definition can silently degrade an execution plan because the optimizer materializes an inline view instead of merging it; how do you detect this in the plan?

**Difficulty:** Expert · **Levels:** SDE-1, SDE-1/2 · [↑ Using Views Effectively](<../../../../01_Databases_and_SQL/Database_Objects/Views/Using_Views_Effectively.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with database views, one common pitfall is that the query optimizer might choose to materialize an inline view rather than merge it into the main execution plan. This can lead to suboptimal performance without any immediate feedback or warning, making it a silent performance degradation. To understand this issue and how to detect it, we need to explore the behavior of the query optimizer and the differences between merged and materialized views.

Firstly, let's clarify what an inline view is. An inline view (also known as derived table) is essentially a temporary result set generated from a subquery that is embedded in another query. The optimizer can decide whether to merge this inline view with the outer query or materialize it into a separate temporary table.

The key difference lies in performance: merging an inline view typically involves executing the subquery just once, and its results are used directly in the main query execution. Materializing the inline view first requires the subquery to be executed separately and stored temporarily, which can introduce additional overhead.

To detect whether the optimizer has chosen to materialize an inline view instead of merging it, we need to analyze the execution plan. This is where the real challenge lies since there might not be any explicit error messages or warnings about performance issues.

**Code:**

```sql
-- Example query with an inline view
SELECT a.col1, b.col2
FROM tableA a
JOIN (SELECT col3, col4 FROM tableB WHERE condition) b ON a.key = b.key;
```

**Example input / output:**

```

To check the execution plan in SQL Server, PostgreSQL, or other databases that support EXPLAIN, you would run:

SQL
Copy
EXPLAIN SELECT a.col1, b.col2
FROM tableA a
JOIN (SELECT col3, col4 FROM tableB WHERE condition) b ON a.key = b.key;

The output will show the execution plan. If an inline view is materialized, you might see something like:

PLAINTEXT
Copy
->  Materialize  (cost=10.56..78.92 rows=374 width=16)
     ->  Subquery Scan on b  (cost=10.56..78.92 rows=374 width=16)
           Filter: (condition)
           ->  Index Only Scan using idx_tableb_key on tableB

In contrast, if the inline view is merged, you will see:

PLAINTEXT
Copy
->  Hash Join  (cost=10.56..78.92 rows=374 width=16)
     Hash Cond: (a.key = b.key)
     ->  Index Only Scan using idx_tableA_key on tableA a
     ->  Subquery Scan on b  (cost=10.56..78.92 rows=374 width=16)
           Filter: (condition)
           ->  Index Only Scan using idx_tableb_key on tableB

The key difference is the presence of Materialize in the plan, which indicates that the optimizer has chosen to materialize rather than merge.
```

**Flow:**

1. Main Query
2. Merge Inline View
3. Execute Subquery Only Once
4. Materialize Inline View
5. Store Temporarily and ExecuteMain Query

