---
type: "practice"
difficulty: "Deep"
---

# Why does a large OFFSET in SQL pagination degrade toward O(n), and what's the complexity-friendly alternative?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Exact vs Approximate Counting](<../../../../08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Exact_vs_Approximate_Counting.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When using SQL pagination with an OFFSET clause, querying data with a large offset value can significantly degrade performance from its intended linear complexity (O(k) where k is the number of rows to fetch) toward O(n), where n is the total number of rows in the table. This is because each query must scan through all previous records before reaching the desired set of results.

To understand this, let's first break down how SQL pagination with OFFSET works and then explore why large offsets can lead to poor performance. Finally, we'll discuss a more complexity-friendly alternative: using an index-based approach with LIMIT and ORDER BY.

#### Triage

What the Question is Asking: The question is probing your understanding of SQL query optimization and pagination techniques.

#### Assumptions

You are familiar with basic SQL operations.
You understand Big O notation for complexity analysis.
You can reason about database indexing strategies.

#### First-Principles Solution
How OFFSET Works

In a typical scenario, the SQL statement might look like this:

SQL
SELECT * FROM users ORDER BY id LIMIT 10 OFFSET 50;

This query instructs the database to return 10 rows from the table starting at row 51. The database engine needs to read all records up to and including the 50th row before it can start returning the requested results.

Why Large OFFSET Degradates Performance
Sequential Scan: When you use a large offset, the database has to scan through the entire table sequentially from the beginning until it reaches the offset. This means that for an offset of 100,000, the database needs to read up to row 100,000 before it starts returning rows.
Inefficiency: The more rows you skip with OFFSET, the higher the computational overhead. Each additional row scanned increases the time complexity linearly.
Alternative: Index-Based Pagination

A more efficient approach is to use an index-based strategy. This involves fetching a small number of rows at a time and using an index to quickly locate the starting point for each fetch. The general idea is:

Use ORDER BY on an indexed column.
Fetch the first row with a specific offset.
Continue fetching subsequent rows from that point onward.

For example, if you are fetching users ordered by their ID and want to start at user 50000, you can use:

SQL
SELECT * FROM users WHERE id >= 50000 ORDER BY id LIMIT 10;

This approach ensures that the database engine only needs to scan a small portion of the table, leading to much better performance for large datasets.

#### Closing Summary
+ Common Mistakes/Follow-Ups
Performance Impact: Large offsets in SQL pagination degrade toward O(n) due to sequential scanning.
Complexity-Friendly Alternative: Using an index-based approach with LIMIT and ORDER BY can keep the complexity at O(k).
Common Mistake: Assuming that large OFFSET values do not affect performance or using them for performance-critical applications.

If asked further, you might be probed on:

How to implement this in your application.

#### Trade-offs
between different pagination strategies.
Using caching mechanisms like Redis to improve response times.

**Comparison:**

| Index-Based Pagination | Sequential Scan |
|------------------------|-----------------|
| O(k)                   | O(n)            |

**Code:**

```sql
-- Example SQL Query with Large OFFSET
SELECT * FROM users ORDER BY id LIMIT 10 OFFSET 50000;

-- Efficient Index-Based Pagination
SELECT * FROM users WHERE id >= 50000 ORDER BY id LIMIT 10;
```

**Example input / output:**

```text
-- Input: Fetch users starting from id 50000 with a limit of 10
SELECT * FROM users WHERE id >= 50000 ORDER BY id LIMIT 10;

-- Output:
+----+----------+
| id | name     |
+----+----------+
| 49998| Zoe      |
| 49999| Mike     |
// ... up to the 10th row starting from 50000
```

**Data:**

```json
{
  "users": [
    {"id": 1, "name": "Alice"},
    {"id": 2, "name": "Bob"},
    // ... up to row 50000,
    {"id": 49998, "name": "Zoe"},
    {"id": 49999, "name": "Mike"}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Is Offset Large?
5. Sequential Scan (O(n))
6. Index-Based Pagination (O(k))
7. High Performance, PoorScalability
8. Good Performance, GoodScalability

