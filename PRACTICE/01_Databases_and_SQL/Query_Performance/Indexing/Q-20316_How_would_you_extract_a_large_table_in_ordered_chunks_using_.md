---
type: "practice"
difficulty: "Deep"
---

# How would you extract a large table in ordered chunks using a keyset (WHERE id > last_id) approach, and what index makes it reliable?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Keyset Pagination](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Keyset_Pagination.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To extract a large table in ordered chunks using a WHERE id > last_id approach, we aim to leverage database indexing effectively to minimize the number of rows processed at once while ensuring data integrity and query performance. The key is choosing an appropriate index that can efficiently support range queries.

#### Triage
What's Being Probed

The question asks about extracting records in ordered chunks using a WHERE id > last_id approach, which is typically used in pagination or cursor-based operations. This requires understanding how indexes work, their impact on query performance, and the trade-offs involved in different database systems.

#### Assumptions
The table has an id column that is a primary key or otherwise unique.
We are working with a relational database system like MySQL or PostgreSQL.
The goal is to efficiently fetch data in chunks without causing too much overhead on the database server.

#### First-Principles Solution

Choose the Right Index:

A B-tree index on the id column will be most suitable for this operation because it supports efficient range scans and ordering.

Query Construction:

Use a query like SELECT * FROM table_name WHERE id > last_id ORDER BY id LIMIT chunk_size; to fetch rows in chunks.

#### Cons
iderations for Performance:

Ensure the index is maintained properly, especially with frequent insertions and deletions.
Monitor lock contention if using row-level locks or other concurrency mechanisms.

Handling Edge Cases:

If last_id reaches the maximum value of the data type used (e.g., INT_MAX for integers), handle it gracefully by either restarting from a known minimum ID or handling an error.
Detailed Steps
1. Index Selection
For a table with many rows, adding a B-tree index on the id column will enable efficient range scans. This ensures that the database can quickly locate the starting point of each chunk without scanning the entire table.
SQL
CREATE INDEX idx_id ON table_name (id);
2. Query Execution
When you run your query with a specific last_id, the database can use the index to find all rows greater than that ID and return them in order.
SQL
SELECT * FROM table_name WHERE id > last_id ORDER BY id LIMIT chunk_size;
The ORDER BY clause ensures that the results are returned in the correct order. This is important for maintaining consistent pagination across multiple fetches.
3. Performance Optimization
Use of an index can drastically reduce the number of rows scanned, but ensure the table statistics are up to date so the optimizer makes good decisions.
Monitor query performance using EXPLAIN plans and adjust last_id handling if needed.

#### Closing Summary

Using a B-tree index on the id column allows efficient extraction of ordered chunks from a large table. The key is to balance the trade-offs between indexing overhead and query performance, ensuring that the chosen approach scales well with the data size and usage patterns.

#### Common Mistakes
and Follow-Ups
Ignoring Index Maintenance: Failing to maintain the index properly can lead to degraded query performance.
Misusing ORDER BY or Lack of It: Without an order clause, results might not be consistent across different queries.
Incorrect Chunk Size Handling: Not handling the edge case where last_id reaches the maximum value can cause issues.

These considerations are crucial for ensuring reliable and efficient data extraction in large-scale applications.

**Comparison:**

    | Column      | Type         |
    |-------------|--------------|
    | id          | BIGINT       |
    | order_id    | VARCHAR(255) |
    | customer_id | INT          |

**Code:**

```sql
-- Example SQL to create an index on 'id'
CREATE INDEX idx_id ON table_name (id);

-- Query to fetch next chunk of records
SELECT * FROM table_name WHERE id > last_id ORDER BY id LIMIT 100;
```

**Example input / output:**

```text
-- Example input for the query
SET @last_id = 54321;
SET @chunk_size = 100;

-- Query execution
SELECT * FROM orders WHERE id > 54321 ORDER BY id LIMIT 100;
```

**Data:**

```json
{
  "table_name": "orders",
  "last_id": 54321,
  "chunk_size": 100
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Is index on 'id' present?
5. Use index for efficient rangescan
6. Create index on 'id'
7. Execute query with ORDER BYand LIMIT clauses
8. Create INDEX idx_id ONtable_name (id)
9. Fetch next chunk of records

