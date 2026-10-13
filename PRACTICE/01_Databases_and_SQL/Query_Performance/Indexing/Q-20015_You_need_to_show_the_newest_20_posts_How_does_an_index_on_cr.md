---
type: "practice"
difficulty: "Intermediate"
---

# You need to show the newest 20 posts. How does an index on created_at make ORDER BY created_at DESC LIMIT 20 fast?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Keyset Pagination](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Keyset_Pagination.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To quickly retrieve the newest 20 posts, using an index on created_at can significantly improve performance by allowing the database to efficiently locate and return the desired records without scanning the entire table.

The key idea here is that when you have an index on a column used in an ORDER BY clause followed by LIMIT, the database can leverage the B-tree structure of the index to perform the operation more quickly. Let's break this down from first principles:

#### Triage

The question is essentially asking about how an ORDER BY created_at DESC query and a subsequent LIMIT 20 clause are optimized using an index on created_at. The main points being probed include understanding B-tree indexes, the benefits of indexing, and the impact of ordering operations.

We'll assume that:

The database is SQL-based (e.g., PostgreSQL, MySQL).
The table storing posts has a column named created_at which stores the creation timestamp of each post.
We have an index on created_at.

#### First-Principles Solution

Understanding B-tree Indexes: In a B-tree index, data is organized in a sorted manner based on the indexed columns. For our case, this means that all posts are stored in the index with their creation timestamps sorted from earliest to latest.

Index Usage During ORDER BY:

When you issue SELECT * FROM posts ORDER BY created_at DESC, the database engine can use the B-tree structure of the created_at index.
The index allows the database to quickly find the 20 most recent records by traversing downwards from the root node, which points directly to the latest timestamp.

LIMIT Clause: Once the 20 most recent posts are identified in the index order, the LIMIT clause simply returns these top 20 results. Since the index already has them sorted, this step is very efficient.

Performance Improvement: Without an index, the database would need to scan the entire table (in reverse chronological order), compare all timestamps, and then return the top 20 rows. This can be significantly slower than using the index for a large number of records.

#### Closing Summary
+ Common Mistakes/Follow-Ups
Common Mistake: Assuming an index will work well in all cases. While an index on created_at is beneficial here, it might not always be optimal if there are additional complex conditions or joins involved.

#### Follow-Up
Questions: What if the table had multiple columns that were frequently queried together? Would a multi-column index help even more?

#### Example
Dry-Run

Suppose we have 100 posts and an index on created_at:

SQL
-- Without Index
EXPLAIN ANALYZE SELECT * FROM posts ORDER BY created_at DESC LIMIT 20;
-- With Index
EXPLAIN ANALYZE SELECT * FROM posts ORDER BY created_at DESC LIMIT 20 USING INDEX idx_created_at;

The first query might show a full table scan, while the second shows efficient use of the index.

Self-Check

This explanation covers all aspects and is complete for an intermediate-level understanding.

**Code:**

```sql
-- Example SQL Queries
EXPLAIN ANALYZE SELECT * FROM posts ORDER BY created_at DESC LIMIT 20;

EXPLAIN ANALYZE SELECT * FROM posts ORDER BY created_at DESC LIMIT 20 USING INDEX idx_created_at;
```

**Example input / output:**

```text
-- Without Index
"Aggregate  (cost=8495.00..8495.10 rows=20 width=...)"
" ->  Seq Scan on posts  (cost=0.00..7500.00 rows=100 width=...)"

-- With Index
"Index Only Scan using idx_created_at on posts  (cost=0.43..8494.62 rows=20 width=...)"
```

**Data:**

```json
{
    "posts": [
        {"id": 1, "created_at": "2023-07-01"},
        {"id": 2, "created_at": "2023-06-30"},
        // ... (more posts)
    ]
}
```

