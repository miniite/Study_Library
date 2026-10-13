---
type: "practice"
difficulty: "Expert"
---

# A query is fast for 99% of users but a 'power user' with 2M rows in a child table times out. How do you design indexes and query shape to be robust to this data-distribution outlier?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Index Tuning in Practice](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Index_Tuning_in_Practice/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with a scenario where a query is fast for 99% of users but significantly slower or even times out for a 'power user' (in this case, one with 2M rows in a child table), the key challenge is to design an indexing and querying strategy that can handle both typical use cases and extreme outliers. This requires understanding the query patterns, data distribution, and performance bottlenecks.

#### Triage
What's Being Probed

The interviewer is probing your ability to:

Analyze Query Performance: Understand how different parts of a query contribute to its execution time.
Design Indexes: Create appropriate indexes that can handle both common and rare use cases efficiently.
Optimize Queries: Formulate queries in such a way that they leverage the indexes effectively.

#### Assumptions
The database is using SQL (could be PostgreSQL, MySQL, or any other relational database).
We are dealing with JOIN operations between tables where one table has 2M rows.
Common usage patterns are well understood and typical user queries involve smaller datasets.
There are no other significant bottlenecks like network latency or hardware limitations.

#### First Principles Solution

Understand the Query: Break down the query to identify what it is doing, especially how it interacts with the 2M-row table. Identify which columns are being accessed and in what order.

Analyze Data Distribution: Use tools like EXPLAIN (in PostgreSQL) or EXPLAIN ANALYZE to see where the performance bottleneck lies. This will help identify whether the issue is primarily around I/O, CPU, or memory.

Indexing Strategy:

Composite Indexes: For columns that are frequently used together in WHERE clauses and JOIN conditions, create composite indexes.
Covering Indexes: If a query can be executed entirely within an index (i.e., no need to fetch data from the table), use covering indexes. This reduces I/O operations.
Partial Indexes: If certain rows are more frequently accessed or are outliers, consider partial indexes that only apply to specific subsets of data.

Query Optimization:

Use INNER JOIN Instead of LEFT JOIN: Unless necessary, avoid LEFT JOINs as they can be slower due to the nature of their operation.
Limit Clause: Ensure that the query includes a LIMIT clause if there is an upper bound on the number of rows needed.
Filter Early: Use conditions in the WHERE clause to filter out non-matching rows early in the execution plan.

Query Parallelization: If possible, break down complex queries into smaller sub-queries and use parallel processing techniques to distribute the load across multiple nodes or cores.

Monitoring and Feedback Loop:

Continuously monitor query performance using APM tools.
Implement a feedback loop where user behavior data is collected and used to refine indexes and queries over time.

#### Closing Summary

In summary, designing an index and optimizing queries for handling both common and rare use cases involves a multi-faceted approach. By understanding the specific patterns of your 'power users' and using tools like EXPLAIN to identify bottlenecks, you can create effective indexing strategies that improve overall performance.

#### Common Mistakes/Follow-Ups
Mistake: Over-indexing or creating redundant indexes.

#### Follow-Up
Always ensure that each index serves a specific purpose and does not cause unnecessary overhead during inserts, updates, and deletes.
Mistake: Ignoring the distribution of data in the query.

#### Follow-Up
Use statistical analysis to understand the distribution of your data and tailor indexing strategies accordingly.
Mistake: Not considering parallel execution or distributed queries for very large datasets.

#### Follow-Up
Evaluate if distributing the workload across multiple nodes can provide significant performance improvements.

**Comparison:**

```
Parent Table
parent_id
Child Table
col1 > 100 Filter
LIMIT 100
```

**Code:**

```sql
-- Example of a composite index
CREATE INDEX idx_child_table ON child_table (col1, col2);

-- Example of a partial index
CREATE INDEX idx_partial ON child_table (col3) WHERE col4 = 'specific_value';

-- Example of using EXPLAIN ANALYZE to understand query execution plan
EXPLAIN ANALYZE SELECT * FROM parent_table JOIN child_table ON parent_table.id = child_table.parent_id WHERE child_table.col1 > 100 LIMIT 100;
```

**Example input / output:**

```text
Query: SELECT * FROM parent_table JOIN child_table ON parent_table.id = child_table.parent_id WHERE child_table.col1 > 100 LIMIT 100;
EXPLAIN ANALYZE Output:
| QUERY PLAN                                                                 |
|---------------------------------------------------------------------------|
| Limit  (cost=25.34..25.36 rows=1 width=89)                                 |
|   ->  Hash Join  (cost=7.56..43.34 rows=200 width=89)                      |
|         Hash Cond: (child_table.parent_id = parent_table.id)               |
|         ->  Index Scan using idx_child_table on child_table                |
|                  Filter: (col1 > 100)                                      |
|         ->  Bitmap Heap Scan on parent_table                              |
|                  Recheck Cond: (id = child_table.parent_id)                |
```

**Data:**

```json
{
  "parent_table": {
    "columns": ["id", "name"],
    "rows": [1, "Alice", 2, "Bob"]
  },
  "child_table": {
    "columns": ["id", "parent_id", "col1", "col2", "col3"],
    "rows": [
      { "id": 1001, "parent_id": 1, "col1": 50, "col2": "value1", "col3": "specific_value" },
      { "id": 1002, "parent_id": 1, "col1": 75, "col2": "value2", "col3": "another_value" }
    ]
  }
}
```

**Flow:**

1. ascii
2. +--------------------------------------------------------------------+
3. | Start                                                              |
4. +--------------------------------------------------------------------+
5. |                                                        |
6. v                                                        |
7. +--------------------------------------------------------------------+
8. | Analyze query and identify bottlenecks using EXPLAIN ANALYZE       |
9. +--------------------------------------------------------------------+
10. |                                 +------------------------+
11. v                                 | Yes                   |
12. +--------------------------------------------------------------------+  |
13. | Identify data distribution patterns                                |  |
14. +--------------------------------------------------------------------+  |
15. |                                 | No                    |
16. v                                 +------------------------+
17. +--------------------------------------------------------------------+
18. | Design composite, covering, and partial indexes                     |
19. +--------------------------------------------------------------------+
20. |                                 +------------------------+
21. v                                 | Yes                   |
22. +--------------------------------------------------------------------+  |
23. | Optimize queries for parallel execution                            |  |
24. +--------------------------------------------------------------------+  |
25. |                                 | No                    |
26. v                                 +------------------------+
27. +--------------------------------------------------------------------+
28. | Monitor performance and refine strategies based on feedback        |
29. +--------------------------------------------------------------------+

