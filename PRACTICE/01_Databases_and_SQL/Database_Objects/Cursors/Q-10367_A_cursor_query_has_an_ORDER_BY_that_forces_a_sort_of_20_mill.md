---
type: "practice"
difficulty: "Deep"
---

# A cursor query has an ORDER BY that forces a sort of 20 million rows before the first fetch returns — how do you make the first row appear faster?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Bulk Collect and FORALL](<../../../../01_Databases_and_SQL/Database_Objects/Cursors/Bulk_Collect_and_FORALL.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To optimize the performance of a cursor query with an ORDER BY clause, particularly when fetching the first row from a large dataset (20 million rows), we need to address the sorting bottleneck. Here's a step-by-step approach:

Understand the Problem: The current query forces a full sort of 20 million rows before returning any results, which is inefficient.
Identify Optimizations: We can use indexing and partitioning strategies to improve performance.

In this solution, we will leverage an index that matches the ORDER BY clause to speed up the initial fetch operation.

**Code:**

```sql
-- Assuming a table named 'large_table' with columns id (primary key), some_data, etc.
CREATE INDEX idx_large_table_order_by ON large_table(some_column) -- Replace 'some_column' with your actual column name
```

**Example input / output:**

```text
Input:
- Query: SELECT * FROM large_table ORDER BY some_column LIMIT 1

Expected Output:
- First row of the ordered data
```

