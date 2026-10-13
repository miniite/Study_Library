---
type: "practice"
difficulty: "Deep"
---

# How does the LIMIT and OFFSET pagination work and why is OFFSET slow for large tables?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Materialized Views vs Incremental Aggregates](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Materialized_Views_vs_Incremental_Aggregates.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand how LIMIT and OFFSET pagination works in PostgreSQL, we first need to grasp the fundamental concepts of both clauses. In a nutshell, LIMIT restricts the number of records returned by a query, while OFFSET skips a specified number of rows before starting to return records. However, using OFFSET with large tables can lead to performance issues because it involves scanning through all previous rows to skip over them.

#### Triaging the Question

The question is asking for an explanation of how LIMIT and OFFSET pagination works in PostgreSQL, specifically focusing on why OFFSET can be slow when dealing with large datasets. The interviewer might be probing your understanding of SQL query execution plans, index usage, and optimization techniques.

#### First-Principles Solution

Let's break this down step by step:

Understanding the Query Execution:

When a LIMIT clause is used without an OFFSET, PostgreSQL can use indexes to efficiently retrieve the required rows.
However, when OFFSET is involved, PostgreSQL must first skip over all rows before the specified offset.

How OFFSET Works Internally:

#### Cons
ider a query like SELECT * FROM users LIMIT 10 OFFSET 5000. To execute this, PostgreSQL needs to scan the entire table up to the 5000th row and then return the next 10 rows.
This means that for large tables, the performance degrades significantly because of the unnecessary scanning.

Performance Impact:

For a table with N rows, PostgreSQL has to scan at least offset + limit rows to retrieve the final result set.
In the worst case, where offset is close to or equal to N, the query effectively scans all rows in the table.

Alternative Solutions:

Use a more efficient pagination method like Cursor-based Pagination or Keyset Pagination.

#### Cons
ider using an index on the ordering column if your dataset allows for it.
For very large datasets, consider splitting the data into smaller chunks and using LIMIT and OFFSET only within those chunks.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, while LIMIT is straightforward, OFFSET can be problematic for large tables because of its inefficiency in skipping rows. To mitigate this, alternative pagination methods should be considered. Key takeaways include understanding the query execution plan and exploring indexes or database-specific optimizations.

#### Common mistakes
often involve not recognizing the performance implications of OFFSET with large datasets and overlooking more efficient alternatives like cursor-based pagination.

For a deeper dive, common follow-ups could include:

How index usage can affect performance.

#### Example
s of when keyset pagination might be applicable versus cursor-based.
Differences between various database systems in handling large table pagination.

**Code:**

```sql
-- Example query using LIMIT and OFFSET
SELECT * FROM users LIMIT 10 OFFSET 5000;
```

