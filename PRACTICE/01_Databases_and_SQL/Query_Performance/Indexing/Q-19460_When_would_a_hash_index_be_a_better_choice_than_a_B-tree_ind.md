---
type: "practice"
difficulty: "Deep"
---

# When would a hash index be a better choice than a B-tree index?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ B-tree and BRIN Indexes](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/B-tree_and_BRIN_Indexes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

Hash indexes and B-tree indexes each have their strengths, making them suitable for different scenarios in database indexing. A hash index is generally faster for exact match queries but less efficient for range scans. Conversely, a B-tree index excels at handling range queries efficiently but can be slower for exact match searches.

#### Triaging the Question

The question asks when a hash index would be a better choice than a B-tree index. To properly answer this, we need to understand both types of indexes and their performance characteristics in various query scenarios.

Key Assumptions:
We are comparing hash indexes with B-tree indexes.
The context is a relational database management system (RDBMS) environment.
Both types of queries involve searching for exact matches or ranges within the indexed columns.
Detailed Solution from First Principles:

Understanding Hash Indexes:

A hash index stores data using a hash function that maps keys to positions in an array. This makes lookups very fast, as they are typically O(1) operations.
However, hash indexes do not support range queries (e.g., WHERE column BETWEEN value1 AND value2 or column >= value). They are most useful for equality and inequality comparisons (=, !=) on specific columns.

Understanding B-Tree Indexes:

A B-tree index organizes data in a tree structure, allowing efficient searching, insertion, and deletion operations.
B-trees support both exact match queries (using the = operator) and range scans (>=, <), making them versatile for a wide variety of query patterns.

When to Choose Hash Indexes:

Exact Match Queries: If you primarily need to perform equality checks, hash indexes can be significantly faster due to their O(1) lookup time.
Data with Unique Keys: In scenarios where the indexed column(s) are highly unique (e.g., primary keys), hash indexes offer consistent performance without the overhead of maintaining a balanced tree structure.

When B-Tree Indexes Are Superior:

Range Queries: For situations involving range searches, such as filtering by date ranges or numerical intervals, B-tree indexes provide efficient and reliable results.
Complex Queries: When dealing with complex queries that involve multiple conditions or joins, the hierarchical structure of B-trees can handle them more effectively.

#### Trade-offs

While hash indexes excel in single key equality checks, they require more memory to store the entire index. This is because every value needs to be mapped to a bucket.
B-tree indexes, on the other hand, are space-efficient but incur higher overhead for maintaining balance and handling range queries.

Practical Example:

#### Cons
ider a database table of users with an id column as the primary key and a created_at timestamp. For retrieving user data by ID (e.g., SELECT * FROM users WHERE id = 12345), a hash index on id would be optimal due to its speed.
Conversely, for finding all users created after a certain date (SELECT * FROM users WHERE created_at > '2023-01-01'), a B-tree index on created_at would perform better.

#### Closing Summary

In summary, hash indexes are preferable when the primary use case involves exact match queries and the data is highly unique. On the other hand, B-tree indexes offer more flexibility and performance in scenarios requiring range scans or complex query conditions. The choice between these two should be based on the specific requirements of your application.

#### Common Mistakes/Follow-Ups
Mistake: Overestimating the frequency of exact match queries compared to range scans.

#### Follow-Up
Discussing alternative indexing strategies like composite indexes, partial indexes, or using covering indexes for performance optimization in complex scenarios.

**Comparison:**

| Query Type       | Exact Match Queries | Range Queries |
|------------------|---------------------|---------------|
| Hash Index       | Fast (O(1))         | Slow          |
| B-tree Index     | Fast                | Efficient     |

**Code:**

```sql
-- Example SQL for creating a hash index and B-tree index

CREATE INDEX idx_users_id ON users (id);
CREATE INDEX idx_users_created_at ON users (created_at);
```

**Example input / output:**

```text
-- Example SQL queries and results

-- Hash Index Usage (Exact Match)
EXPLAIN SELECT * FROM users WHERE id = 1;
-- Output: Hash index scan

-- B-Tree Index Usage (Range Scan)
EXPLAIN SELECT * FROM users WHERE created_at >= '2023-01-01';
-- Output: Bitmap index scan or similar
```

**Data:**

```json
{
  "users": [
    { "id": 1, "name": "Alice", "created_at": "2023-01-01" },
    { "id": 2, "name": "Bob", "created_at": "2023-02-01" }
  ]
}
```

**Flow:**

1. Exact Match Query
2. Hash Index
3. B-Tree Index
4. Range Query
5. B-Tree Index
6. Hash Index (Slow)

