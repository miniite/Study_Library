---
type: "practice"
difficulty: "Deep"
---

# How would you make an existing OFFSET-based paginated endpoint fast on deep pages while the frontend still expects page numbers during the migration?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Offset Pagination Pitfalls](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Offset_Pagination_Pitfalls.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To optimize a database query to support efficient pagination using OFFSET without sacrificing performance, especially as we move towards modernizing an existing system that relies heavily on OFFSET, we need to consider several strategies and trade-offs. This problem is fundamentally about balancing the use of index-based access with the limitations of LIMIT OFFSET. The key challenge lies in maintaining the integrity of page numbers while ensuring fast queries for deep pages.

#### Triage

The question revolves around optimizing a paginated query that currently uses OFFSET but will eventually shift towards more efficient pagination methods, such as using an indexed column. We need to understand the current system and identify potential bottlenecks. The main goal is to ensure that deep pages are fast while keeping page numbers consistent for the frontend.

#### Assumptions
The existing query uses a LIMIT OFFSET pattern.
The database schema includes an index on the relevant column used for pagination.
The application expects specific page numbers, which must be preserved during optimization.
We can introduce temporary measures to minimize disruptions while gradually implementing better solutions.
Step-by-Step Solution

Understand Current Query Structure: Consider a query structure like:

SQL
SELECT * FROM users ORDER BY id LIMIT 20 OFFSET 300;

This returns the 301st to 320th rows based on id.

Indexing Strategy: To optimize this, we need an index that can quickly jump to specific ranges of data without relying solely on OFFSET. For example:

Use a composite index on (page_number, id) if the page_number is known.
Alternatively, use an index like B-tree or bitmap index on the pagination column.

Materialized Views: Create materialized views that store precomputed segments of data. This can be especially useful for deep pages:

SQL
CREATE MATERIALIZED VIEW page_100_to_120 AS
SELECT * FROM users ORDER BY id LIMIT 20 OFFSET 980;

Then, the query to get page 100 could simply be:

SQL
SELECT * FROM page_100_to_120;

Partitioning: If partitioning is supported in your database (e.g., PostgreSQL), create partitions based on page_number or other ranges:

SQL
CREATE TABLE users_partitioned PARTITION BY RANGE(id);

This allows the query to skip large portions of data, reducing I/O.

Cursor-based Pagination: Gradually migrate to cursor-based pagination if supported by your database system:

SQL
SELECT * FROM users ORDER BY id LIMIT 20 OFFSET 300 FOR UPDATE;

This uses a cursor to maintain state between queries, ensuring consistent results for deep pages.

Caching: Implement caching mechanisms using in-memory databases like Redis or Memcached. Store frequently accessed pages and invalidate them based on page numbers:

PYTHON
cache.get('page_100')

This reduces the load on the database for subsequent requests to the same page.

#### Closing Summary

By combining these strategies, we can significantly improve the performance of deep pagination queries while maintaining the requirement that the frontend expects specific page numbers. The key is to balance indexing with caching and gradually introduce more efficient techniques like materialized views or cursor-based pagination.

#### Common Mistakes/Follow-Ups
Over-indexing: Creating too many indexes can degrade write performance.
Choosing the Right Index: Not selecting an appropriate index can lead to suboptimal query performance.
Caching Invalidation: Incorrect caching strategies can result in stale data being served.
Migrating Too Quickly: A sudden transition from OFFSET to more advanced techniques without a fallback plan can cause downtime.

**Comparison:**

```
Yes
No
Yes
No
Query Request
Is Offset Query?
Execute with OFFSET
Materialized View Exists?
Fetch from Materialized View
Create Materialized View, thenFetch
```

**Code:**

```sql
-- Example of creating a materialized view
CREATE MATERIALIZED VIEW page_100_to_120 AS
SELECT * FROM users ORDER BY id LIMIT 20 OFFSET 980;

-- Query using the materialized view
SELECT * FROM page_100_to_120;
```

**Example input / output:**

```

Input: Request for page 100

Initial Query (using OFFSET):

SQL
Copy
SELECT * FROM users ORDER BY id LIMIT 20 OFFSET 980;

Optimized Query (using materialized view):

SQL
Copy
SELECT * FROM page_100_to_120;

Output: Data for page 100
```

**Data:**

```json
{
  "page_number": 100,
  "user_ids": [1050, 1051, ..., 1070]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Query Request
6. Is Offset Query?
7. Execute with OFFSET
8. Materialized View Exists?
9. Fetch from Materialized View
10. Create Materialized View, thenFetch

