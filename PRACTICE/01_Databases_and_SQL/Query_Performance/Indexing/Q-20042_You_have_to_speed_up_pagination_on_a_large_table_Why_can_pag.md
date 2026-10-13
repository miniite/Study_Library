---
type: "practice"
difficulty: "Intermediate"
---

# You have to speed up pagination on a large table. Why can page 5000 with OFFSET be slow and what is a simpler alternative?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Offset Pagination Pitfalls](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Offset_Pagination_Pitfalls.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

Page 5000 using OFFSET can be slow because it requires the database to scan through all preceding rows, which becomes prohibitively expensive as the table grows. A simpler and more efficient alternative is to use a combination of indexes and LIMIT/OFFSET pagination.

#### Triage
The Question and What's Being Probed

The question asks you to explain why using OFFSET for pagination on a large table can be slow and to propose an alternative method. This problem is common in database development, especially when dealing with large datasets where performance matters. The interviewer is likely looking for your understanding of query optimization techniques and the trade-offs between different approaches.

Step-by-Step Solution from First Principles
Understanding OFFSET Pagination

When you use OFFSET to implement pagination, the SQL query needs to skip a certain number of rows and then return the next set. For example:

SQL
SELECT * FROM large_table LIMIT 10 OFFSET (5000 - 1) * 10;

This means that the database engine has to read through all records from the first row up to the OFFSET position before it can start returning results. This is inefficient for two main reasons:

Time Complexity: The time taken increases linearly with the number of rows skipped, making large OFFSET values very slow.
Resource Usage: The database must keep track of all read and ignored rows, which consumes memory and CPU.
A Better Approach: Indexes and Covering Indexes

Instead of using OFFSET, we can leverage indexes to directly jump to the desired page. We achieve this by:

Creating an appropriate index on a column used in the query.
Using the index to quickly locate the starting point for our desired rows.

For example, if you have a large table with millions of records and frequently need to paginate based on user_id, creating an index on user_id can significantly speed things up:

SQL
CREATE INDEX idx_user_id ON large_table(user_id);
Implementing Covering Indexes

A covering index is one that includes all the columns needed by a query. This allows the database to satisfy the query entirely from the index itself, without needing to access the underlying table rows. For instance:

SQL
CREATE INDEX idx_user_id ON large_table(user_id, column1, column2);

In this case, once you find the starting point using the user_id index, the database can directly serve the required columns from the index.

#### Example
Query with Indexes

Let's say we want to fetch page 500 for a table where each page contains 10 rows, and we have an index on user_id:

SQL
-- Find the starting row ID
WITH start_row AS (
    SELECT MIN(user_id) FROM large_table WHERE user_id > (499 * 10)
),
row_ids AS (
    SELECT user_id FROM large_table WHERE user_id BETWEEN (SELECT user_id FROM start_row) AND ((499 + 1) * 10 - 1) ORDER BY user_id
)
-- Fetch the actual rows from the table using the row IDs
SELECT * FROM large_table WHERE user_id IN (SELECT user_id FROM row_ids);

#### Closing Summary
and Common Mistakes/Follow-Ups

Using indexes to optimize pagination is more efficient because it reduces the amount of data the database needs to scan. The key takeaway is that OFFSET should be avoided for large datasets, as it leads to poor performance due to its linear-time complexity.

#### Common mistakes
include:

Not considering indexing: Always check if creating an appropriate index can speed up queries.
Overusing LIMIT and OFFSET: These are often slow for pagination, especially with large tables.

#### Follow-ups
might involve discussing how different database engines handle this differently, or exploring more advanced techniques like using cursors in some databases (e.g., PostgreSQL).

Depth + Language Match the Level

As a fresher to an intermediate experience level, you should focus on explaining the concepts clearly and providing examples. For a principal engineer role, delve into more detailed performance metrics, database engine optimizations, and best practices for large-scale systems.

**Comparison:**

| Column | Type    |
|--------|---------|
| id     | integer |
| user_id| integer |
| column1| text    |
| column2| text    |
| Index Name   | Type  | Columns       |
|--------------|-------|---------------|
| idx_user_id  | btree | user_id       |

**Code:**

```

CREATE INDEX idx_user_id ON large_table(user_id);

CODE
Copy
```sql
WITH start_row AS (
    SELECT MIN(user_id) FROM large_table WHERE user_id > (499 * 10)
),
row_ids AS (
    SELECT user_id FROM large_table WHERE user_id BETWEEN (SELECT user_id FROM start_row) AND ((499 + 1) * 10 - 1) ORDER BY user_id
)
SELECT * FROM large_table WHERE user_id IN (SELECT user_id FROM row_ids);
```

**Example input / output:**

```text
-- Create an index on user_id
CREATE INDEX idx_user_id ON large_table(user_id);

-- Query to get page 500 with 10 rows per page using the index
WITH start_row AS (
    SELECT MIN(user_id) FROM large_table WHERE user_id > (499 * 10)
),
row_ids AS (
    SELECT user_id FROM large_table WHERE user_id BETWEEN (SELECT user_id FROM start_row) AND ((500 + 1) * 10 - 1) ORDER BY user_id
)
SELECT * FROM large_table WHERE user_id IN (SELECT user_id FROM row_ids);
```

**Data:**

```json
{
    "table_name": "large_table",
    "columns": ["id", "user_id", "column1", "column2"],
    "index": {
        "idx_user_id": {"type": "btree", "on_columns": ["user_id"]}
    }
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Is there an index?
5. Use the index to find thestarting row
6. Use the row IDs to fetch actualdata from table
7. Use OFFSET for pagination,which is slow andresource-intensive

