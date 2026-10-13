---
type: "practice"
difficulty: "Deep"
---

# A query filters WHERE tags @> ARRAY['x','y'] on a Postgres array column. Explain how a GIN index serves it and when it stops helping (e.g. very common tags).

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ JSON and Text Search Indexes](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/JSON_and_Text_Search_Indexes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary:

When dealing with queries filtering an array column like tags @> ARRAY['x','y'] in PostgreSQL, a GIN index is crucial because it allows efficient lookups and reduces the need for full table scans. However, this advantage diminishes as the frequency of common tags increases.

#### Triage

The question asks us to explain how a GIN (Generalized Inverted Index) serves queries filtering an array column in PostgreSQL and under what conditions it may stop being effective. We'll start by understanding what a GIN index is and then delve into its benefits and limitations, particularly when common tags are frequently queried.

#### First Principles Solution
1. Understanding the Query:

The query WHERE tags @> ARRAY['x','y'] checks if an array column contains both 'x' and 'y'. In PostgreSQL, this operator works well with GIN indexes on array columns.

2. What is a GIN Index?

A GIN index in PostgreSQL is designed to efficiently handle queries involving array and text data types. It stores the inverted index structure for elements of the indexed column, making lookups faster by reducing the number of rows that need to be scanned.

3. How GIN Indexes Serve the Query:
Efficient Lookup: A GIN index on tags allows PostgreSQL to quickly find all rows where tags contains both 'x' and 'y'. Without an index, this would require a full table scan.
Reduced Overhead: The GIN index helps by reducing the number of disk I/O operations needed.
4. When Does It Stop Helping?

The effectiveness of the GIN index can degrade when:

High Frequency Tags: If 'x' and 'y' are common tags, many rows will match the query condition. This increases the index size and makes it less efficient.
Index Degeneracy: As more rows contain the common tags, the index becomes less selective, meaning that a lot of full table scans are still required.
5. Example Scenario:

Suppose in a dataset of posts, most posts have 'x' and 'y' as tags. The GIN index will help initially but might not be cost-effective when over 70% of the rows contain both tags.

#### Closing Summary
+ Common Mistakes/Follow-Ups:
Common Mistake: Assuming that high cardinality in array elements always benefits from a GIN index without considering query frequency.

#### Follow-Up
Question: How would you design an indexing strategy if common tags are frequent, yet rare tags need to be efficiently queried as well?
Common Edge Cases and Trade-offs:
Selective Indexes: Consider creating multiple indexes on different tag combinations to handle both common and rare queries effectively.
Bitmap Indexes: For very large datasets with common tags, bitmap indexes might offer better performance.

#### Example
Input/Output:
SQL
-- Creating a table and index
CREATE TABLE posts (
  id SERIAL PRIMARY KEY,
  title VARCHAR(255),
  tags TEXT[]
);
CREATE INDEX idx_tags ON posts USING GIN (tags);

-- Inserting sample data
INSERT INTO posts (title, tags) VALUES ('Post with x', ARRAY['x']);
INSERT INTO posts (title, tags) VALUES ('Post with y', ARRAY['y']);
INSERT INTO posts (title, tags) VALUES ('Post with both', ARRAY['x','y']);

-- Query performance with and without index
EXPLAIN ANALYZE SELECT * FROM posts WHERE tags @> ARRAY['x','y']; -- Using GIN index

EXPLAIN ANALYZE SELECT * FROM posts WHERE tags @> ANY(ARRAY[ARRAY['x'], ARRAY['y']]); -- Without GIN, full table scan

Diagrams:
Yes
No
GIN Index
Is Common Tag Frequent?
Index Degenerates
Index Remains Effective
Frequent Full Table ScansRequired
Efficient Lookup
Self-Check:

This explanation covers the core concepts and provides a thorough analysis of GIN index usage.

**Code:**

```sql
CREATE TABLE posts (
  id SERIAL PRIMARY KEY,
  title VARCHAR(255),
  tags TEXT[]
);
CREATE INDEX idx_tags ON posts USING GIN (tags);

INSERT INTO posts (title, tags) VALUES ('Post with x', ARRAY['x']);
INSERT INTO posts (title, tags) VALUES ('Post with y', ARRAY['y']);
INSERT INTO posts (title, tags) VALUES ('Post with both', ARRAY['x','y']);

EXPLAIN ANALYZE SELECT * FROM posts WHERE tags @> ARRAY['x','y']; -- Using GIN index
EXPLAIN ANALYZE SELECT * FROM posts WHERE tags @> ANY(ARRAY[ARRAY['x'], ARRAY['y']]); -- Without GIN, full table scan
```

**Example input / output:**

```text
-- Query performance with GIN index
QUERY PLAN:
Bitmap Heap Scan on posts  (cost=1.80..25.47 rows=3 width=60)
Recheck Cond: ((tags @> '{x,y}'::text[]) AND (...))
->  Bitmap And  (cost=1.80..1.80 rows=1 width=0)
->  Bitmap Index Scan on idx_tags  (cost=0.00..1.79 rows=3 width=0)
Index Cond: ((tags @> '{x}'::text[]) AND (...))
->  Bitmap Index Scan on idx_tags  (cost=0.00..0.80 rows=1 width=0)
Index Cond: (tags @> '{y}'::text[])

-- Without GIN, full table scan
QUERY PLAN:
Seq Scan on posts  (cost=0.00..23.60 rows=14 width=60)
Filter: ((tags @> ANY ('{x,y}'::text[])) AND (...))
```

**Data:**

```json
[
  {
    "id": 1,
    "title": "Post with x",
    "tags": ["x"]
  },
  {
    "id": 2,
    "title": "Post with y",
    "tags": ["y"]
  },
  {
    "id": 3,
    "title": "Post with both",
    "tags": ["x","y"]
  }
]
```

