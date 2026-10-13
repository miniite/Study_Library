---
type: "practice"
difficulty: "Deep"
---

# Why does OFFSET-based pagination over a huge export table get slower on later pages, and what simpler approach works for chunked extraction?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Offset Pagination Pitfalls](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Offset_Pagination_Pitfalls.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

OFFSET-based pagination can become inefficient as you move to later pages due to how it works under the hood. Essentially, with each page request, the database has to scan through all the rows before the offset point and then return the next set of rows. This means that on the first page, only a small portion of the table is scanned, but by the time you reach the 10th or 20th page, almost the entire table has been read into memory, making subsequent queries slower.

The key idea here is to optimize the data extraction process for large tables to avoid inefficient pagination. A simpler and more efficient approach involves using a cursor-based method (or a similar technique) that maintains state between requests, allowing you to directly fetch from where the previous query left off.

To fully understand this issue, let's break it down step by step:

#### Triage

The question is asking why OFFSET-based pagination degrades as you move through more pages and what alternative methods can be used for efficient data extraction. The interviewer might probe your understanding of how databases work with large tables and the trade-offs between different query optimization techniques.

#### Assumptions
We are dealing with a very large table that needs to be exported or paginated.
The query is being run on a relational database system (like PostgreSQL, MySQL, etc.).

#### First-Principles Solution
Understanding OFFSET-Based Pagination

OFFSET-based pagination works by specifying the number of rows to skip (offset) and the number of rows to return (limit). For example, if you want the 10th page with 50 results each, your query might look something like this:

SQL
SELECT * FROM huge_table LIMIT 50 OFFSET (page_number - 1) * 50;

The problem arises because for every subsequent page, the database must read all rows up to and including the offset point before it can return the requested number of rows.

Performance Degradation

As the offset value increases with each request, the amount of data that needs to be processed also increases. This leads to a quadratic growth in the time required to execute each query as you progress through more pages:

Page 1: Skips 0 rows (fast)
Page 2: Skips 50 rows
Page 3: Skips 100 rows
Page N: Skips ((N - 1) * 50) rows
Cursor-Based Pagination as a Solution

A more efficient approach is to use cursor-based pagination, where you maintain a state (cursor) between requests. This can be implemented in various ways:

Primary Key or Unique Identifier: Use the last row's primary key from the previous page as the starting point.
Row Number Column: If your database supports it, use a ROW_NUMBER() window function to generate unique identifiers for rows.

#### Example
of using cursor-based pagination:

SQL
-- First request: get initial data and store the max_id
SELECT * FROM huge_table WHERE id > (last_max_id) LIMIT 50;

-- Subsequent requests: use the last_max_id from the previous result as the starting point
SELECT * FROM huge_table WHERE id > (last_max_id) LIMIT 50;

#### Trade-offs
and Considerations
Cursor Maintenance: Requires additional logic to manage state between queries.
Performance for Initial Pages: Cursor-based pagination might be slower initially since it requires an initial scan or index lookup.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, OFFSET-based pagination is inefficient for large tables due to its linear search nature. Using a cursor-based approach can significantly improve performance by allowing direct access to the required data without processing unnecessary rows. However, this comes with additional complexity in managing state between requests.

#### Common mistakes
include:

Failing to understand the quadratic growth of time complexity.
Not considering alternative pagination methods that are better suited for large datasets.

#### Follow-ups
might involve discussing specific implementation details or performance optimizations related to cursor-based pagination.

**Code:**

```sql
-- Example SQL query using ROW_NUMBER()
WITH numbered_rows AS (
  SELECT id, ROW_NUMBER() OVER (ORDER BY id) as row_num FROM huge_table
)
SELECT * FROM numbered_rows WHERE row_num > last_max_id LIMIT 50;
```

