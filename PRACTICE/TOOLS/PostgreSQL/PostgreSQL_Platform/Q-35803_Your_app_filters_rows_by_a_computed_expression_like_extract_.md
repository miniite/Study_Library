---
type: "practice"
difficulty: "Intermediate"
---

# Your app filters rows by a computed expression like extract(year from created_at) and it never uses the index; explain why and show two ways to make it index-friendly.

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Indexing Multi-join Queries](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Indexing_Multi-join_Queries.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with filtering rows in PostgreSQL using an expression like EXTRACT(YEAR FROM created_at) without leveraging indexes, you face a common issue: the database cannot use an index on created_at to speed up your query. To fix this, we need to understand why it's not using the index and then explore two effective solutions.

Firstly, let’s break down the problem:

The EXTRACT(YEAR FROM created_at) function creates a computed column that PostgreSQL cannot recognize as an indexed column.
Without an appropriate index, PostgreSQL resorts to scanning the entire table or at least the relevant portion of it, which can be inefficient for large datasets.

To make this query more efficient, we need to ensure that PostgreSQL can use indexes effectively. Here are two strategies:

Strategy 1: Create a Partial Index

One way is to create a partial index on the created_at column, filtering by the year directly:

SQL
CREATE INDEX idx_created_at_year ON your_table (created_at) WHERE EXTRACT(YEAR FROM created_at) = 2023;

This allows PostgreSQL to use an index specifically for queries that filter by the specified year. However, this approach requires knowing the exact value or range of values you are filtering on.

Strategy 2: Create a Function-Based Index

Another robust solution is to create a function-based index:

SQL
CREATE INDEX idx_created_at_year_function ON your_table ((EXTRACT(YEAR FROM created_at)));

This tells PostgreSQL that it can use this computed column directly in the query execution plan. Now, when you run a query like SELECT * FROM your_table WHERE EXTRACT(YEAR FROM created_at) = 2023;, the index will be used to quickly find matching rows.

#### Triage
and Assumptions
Assumption: You are using PostgreSQL version 12 or later.
Objective: Optimize query performance by ensuring efficient use of indexes.

#### Cons
traints: The created_at column is a timestamp or date type.

#### First-Principles Solution

By creating the appropriate index, we can ensure that the database takes advantage of this indexed column. However, there are trade-offs to consider:

Partial Indexes: Limited in flexibility and require knowing specific filter values.
Function-Based Indexes: More flexible but might have overhead due to additional metadata storage.

#### Closing Summary
+ Common Mistakes/Follow-Ups

Creating a function-based index is usually the preferred approach as it provides more flexibility. However, always test both approaches with your actual data to see which performs better in practice. Additionally, consider monitoring query performance and index usage over time to ensure ongoing efficiency.

#### Common mistakes
include:

Forgetting to create an appropriate index.
Overlooking partial indexes when specific filter values are known.
Not testing the effectiveness of different indexing strategies before deployment.

**Code:**

```sql
-- Example of creating a function-based index
CREATE INDEX idx_created_at_year_function ON your_table ((EXTRACT(YEAR FROM created_at)));
```

**Example input / output:**

```

None required here, but an example input-output could be:

SQL
Copy
-- Input
SELECT * FROM your_table WHERE EXTRACT(YEAR FROM created_at) = 2023;

-- Output (using function-based index)
+----+---------------------+...+
| id | created_at          |
+----+---------------------+...+
| 1  | 2023-04-15 14:45:00 |
| 2  | 2023-06-29 17:23:00 |
+----+---------------------+...+
```

