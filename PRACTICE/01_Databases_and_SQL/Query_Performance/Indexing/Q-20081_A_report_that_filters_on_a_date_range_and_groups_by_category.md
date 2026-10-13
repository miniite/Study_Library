---
type: "practice"
difficulty: "Intermediate"
---

# A report that filters on a date range and groups by category is slow. What single composite index would you try first and how would you validate it against EXPLAIN?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Index Tuning in Practice](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Index_Tuning_in_Practice/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the performance issue of a slow report filtering by date range and grouping by category, we need to understand that indexing can significantly speed up query execution times by reducing the number of disk accesses required. For this specific case, a composite index on (date, category) would be our first step. This index will allow the database engine to efficiently scan rows within the specified date range while also leveraging the category column for grouping operations.

#### Triage
and Assumptions
Assumption 1: The report query is complex enough that optimizing the indexing can significantly improve performance.
Assumption 2: The database being used supports composite indexes, as not all databases do (e.g., some NewSQL systems may have different requirements).
Assumption 3: The data distribution is such that a significant number of queries will filter by date and group by category.
Step-by-Step Solution

Identify the Query:

SQL
SELECT category, COUNT(*)
FROM sales
WHERE sale_date BETWEEN '2023-01-01' AND '2023-12-31'
GROUP BY category;

Create a Composite Index:

SQL
CREATE INDEX idx_sales_date_category ON sales (sale_date, category);

Analyze the Query with EXPLAIN: To validate our index choice, we can use the EXPLAIN command to see how the database engine plans to execute the query before and after creating the index.

Dry Run with EXPLAIN:

SQL
EXPLAIN SELECT category, COUNT(*) FROM sales WHERE sale_date BETWEEN '2023-01-01' AND '2023-12-31' GROUP BY category;

Before Index:

The query might perform a full table scan or use an index on sale_date but still require additional operations for grouping.

After Index:

With the composite index, the database should be able to filter by both sale_date and category more efficiently, potentially using a single pass through the index.

#### Closing Summary
+ Common Mistakes/Follow-Ups
Summary: By creating an index on (date, category), we can reduce the number of rows that need to be processed for grouping, thereby improving performance.

#### Common Mistakes
Failing to consider the data distribution and query patterns before indexing.
Creating overly broad indexes that might not provide significant benefits or even degrade performance in certain scenarios (e.g., too many columns in the index).
Not validating the effectiveness of the new index through EXPLAIN output.

#### Follow-Ups

What if the data distribution is uneven?

In such cases, consider partitioning the table by date ranges to further optimize performance.

If there are additional query patterns?

Analyze other common queries and adjust indexes accordingly to cover as many use cases as possible.

**Code:**

```sql
CREATE INDEX idx_sales_date_category ON sales (sale_date, category);
```

**Example input / output:**

```text
-- Before Index
EXPLAIN SELECT category, COUNT(*) FROM sales WHERE sale_date BETWEEN '2023-01-01' AND '2023-12-31' GROUP BY category;

-- After Index
EXPLAIN SELECT category, COUNT(*) FROM sales WHERE sale_date BETWEEN '2023-01-01' AND '2023-12-31' GROUP BY category;
```

