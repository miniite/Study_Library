---
type: "practice"
difficulty: "Deep"
---

# What does 'Using temporary' in EXPLAIN indicate and which operations commonly trigger it?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Reading EXPLAIN](<../../../../TOOLS/MySQL/MySQL_Platform/Reading_EXPLAIN.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When you see Using temporary in an EXPLAIN plan, it means that MySQL is using a temporary table or file to store intermediate results of your query. This typically happens when the optimizer decides that creating a temporary structure will help optimize the execution plan. Operations like subqueries, join operations with certain types of joins (like LEFT JOIN), and complex queries involving ORDER BY can trigger this.

#### Triage
What Is Being Probed

The question is essentially asking you to understand how MySQL optimizes complex queries and when it resorts to using temporary storage. The interviewer might be probing your knowledge on query optimization, understanding of internal workings of the database, and ability to identify performance bottlenecks in SQL execution plans.

#### Assumptions
You are familiar with EXPLAIN output.
You understand basic SQL operations like JOINs, subqueries, etc.
You know about temporary tables/files used by MySQL.

#### First Principles Solution

Understanding Temporary Tables/Files:

A temporary table is a special kind of table that exists only for the duration of a session or transaction and is then automatically dropped when it's no longer needed.
Temporary files are used in scenarios where sorting data larger than available memory requires writing intermediate results to disk.

Common Operations Triggering Using Temporary:

Subqueries: When you use subqueries, especially complex ones involving multiple levels or correlated subqueries, MySQL may need temporary storage to hold intermediate result sets.
Joins with FULL OUTER JOINs and LEFT/RIGHT JOINs: For these types of joins, MySQL might need a temporary table to handle the non-equi-join conditions.
ORDER BY with large datasets: When sorting operations exceed available memory or are too complex for in-memory processing, MySQL may use a temporary file to store intermediate results.

#### Example Scenario
s:

#### Cons
ider a query like SELECT * FROM table1 JOIN (SELECT * FROM table2 WHERE id > 10) ON .... Here, the subquery might trigger Using temporary if it involves sorting or large result sets.
For a FULL OUTER JOIN, MySQL needs to handle missing values in one of the tables. This often requires storing intermediate results temporarily.

#### Closing Summary
+ Common Mistakes/Follow-Ups
Recognizing Using temporary is crucial for performance tuning as it indicates inefficient query execution plans that might benefit from rewriting or indexing.
To mitigate this, consider optimizing your queries by adding appropriate indexes, using derived tables instead of subqueries where possible, and rewriting joins to be more efficient.

#### Common Mistakes
Ignoring Using temporary can lead to poor performance without realizing the underlying reason.
Overlooking simpler optimizations (like creating indexes) before concluding that there’s no other way to improve query performance.

#### Follow-Ups
Can you provide an example of how adding an index could avoid using a temporary table?
How would you optimize a query with Using temporary in its execution plan?

**Code:**

```sql
-- Example SQL Query that might use temporary storage
EXPLAIN SELECT * FROM employees e
JOIN (SELECT * FROM projects p WHERE start_date > '2023-01-01') p
ON e.project_id = p.id;
```

**Example input / output:**

| id | select_type  | table        | type   | possible_keys       | key     | key_len | ref          | rows    | filtered | Extra           |
|----|-------------|--------------|--------|---------------------|---------|---------|--------------|---------|----------|-----------------|
| 1  | SIMPLE      | employees    | ALL    | NULL                | NULL    | NULL    | NULL         | 50000   | 100.00   | Using temporary |
| 1  | MATERIALIZED| <derived2>   | ALL    | NULL                | NULL    | NULL    | NULL         | 50000   | 100.00   | Using filesort  |

**Flow:**

1. No
2. Yes
3. Yes
4. No
5. Start
6. Query Complex?
7. Optimized Query
8. Subquery or Join?
9. Using Temporary
10. In-Memory Processing
11. Check Indexes and RewriteQuery

