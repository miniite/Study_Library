---
type: "practice"
difficulty: "Deep"
---

# A many-to-many tag search 'posts having all of tags A, B, C' is slow. Compare a GIN index on a tag array, a normalized join with grouping, and an inverted-index table.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ JSON and Text Search Indexes](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/JSON_and_Text_Search_Indexes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To tackle the performance issue in a many-to-many tag search 'posts having all of tags A, B, C,' we need to compare three common approaches: using a GIN index on a tag array, performing a normalized join with grouping, and utilizing an inverted-index table. The goal is to determine which method provides the best balance of performance, scalability, and maintainability.

First, let's summarize the problem:

We have posts that can be tagged multiple times.
A query needs to find posts that match all specified tags (e.g., 'A', 'B', 'C').
Current implementation is slow for large datasets.

Now, let’s break down each approach:

1. GIN Index on a Tag Array

#### Overview

A GIN index allows efficient queries on arrays by storing the values in an inverted index structure. This makes it faster to query posts based on tag presence.

#### Assumptions
The database supports GIN indices.
Tags are stored as an array in the posts table.
Steps:
Index Creation:
SQL
CREATE INDEX idx_posts_tags_gin ON posts USING GIN (tags);
Query Execution:
SQL
SELECT * FROM posts WHERE tags @> '["A", "B", "C"]';

#### Pros
Fast for exact matches and containment queries.
Simplifies the schema by avoiding joins.

#### Cons
Can be memory-intensive due to array storage.
May not perform well if there are many unique tags or if tag lists are very long.
2. Normalized Join with Grouping

#### Overview

Normalization separates the tags into a separate table, ensuring that each post-tag relationship is represented by a single record in the join table. This approach allows for more efficient queries using joins and aggregations.

#### Assumptions
The database schema includes a normalized tags table.
There’s a join between posts and post_tags.
Steps:

Schema:

SQL
CREATE TABLE posts (id SERIAL PRIMARY KEY, title TEXT);

CREATE TABLE post_tags (post_id INTEGER REFERENCES posts(id), tag_id INTEGER REFERENCES tags(id));

CREATE TABLE tags (id SERIAL PRIMARY KEY, name TEXT UNIQUE NOT NULL);

Indexing:

SQL
CREATE INDEX idx_post_tags_tag_id ON post_tags (tag_id);

Query Execution:

SQL
SELECT p.* FROM posts p
JOIN post_tags pt ON p.id = pt.post_id
WHERE tag_id IN ('A', 'B', 'C')
GROUP BY p.id HAVING count(DISTINCT pt.tag_id) = 3;

#### Pros
Efficient for complex queries using multiple tags.
Better schema normalization, easier to maintain.

#### Cons
Increased complexity in the schema.
More overhead due to join operations.
3. Inverted-Index Table

#### Overview

An inverted index is a data structure that maps content to identifiers of documents or items that contain the content. This approach can be highly efficient for full-text search but might need additional setup and optimization.

#### Assumptions
The database supports an inverted index.
Tags are stored as individual rows in the tags table, with foreign keys linking to posts.
Steps:

Schema:

SQL
CREATE TABLE tags (id SERIAL PRIMARY KEY, name TEXT UNIQUE NOT NULL);

CREATE TABLE post_tags (post_id INTEGER REFERENCES posts(id), tag_id INTEGER REFERENCES tags(id));

Inverted Index Setup: Typically handled by a full-text search engine like PostgreSQL’s tsvector or Elasticsearch.

Query Execution:

SQL
SELECT p.* FROM posts p
JOIN post_tags pt ON p.id = pt.post_id
WHERE tag_id IN ('A', 'B', 'C');

#### Pros
High performance for full-text search.
Can handle complex query patterns efficiently.

#### Cons
Requires additional setup and maintenance.
Less flexible for simple queries compared to GIN or normalized joins.

#### Closing Summary

For the given problem, a GIN index on tags is suitable if simplicity and ease of implementation are prioritized. The normalized join approach offers better performance for complex queries but at the cost of schema complexity and additional overhead. An inverted-index table can be more efficient in scenarios requiring full-text search capabilities but requires specialized setup.

#### Common Mistakes/Follow-ups
Ignoring Index Maintenance: Ensure that indices are regularly maintained to avoid degradation.
Overlooking Schema Normalization Benefits: Over-normalizing the schema might not always be necessary, especially if GIN provides sufficient performance.
Full Text Search Misconceptions: Inverted-index tables should only be considered if full-text search features are required.

**Comparison:**

| Post | Tags |
|------|------|
| P1   | ["A", "B"]  |
| P2   | ["A", "B", "C"]  |
| P3   | ["A", "D"]  |
| Tag | Posts |
|-----|-------|
| A   | [P1, P2]  |
| B   | [P1, P2]  |
| C   | [P2]     |

**Code:**

```sql
-- GIN Index Creation
CREATE INDEX idx_posts_tags_gin ON posts USING GIN (tags);

-- Normalized Join with Grouping
SELECT p.* FROM posts p
JOIN post_tags pt ON p.id = pt.post_id
WHERE tag_id IN ('A', 'B', 'C')
GROUP BY p.id HAVING count(DISTINCT pt.tag_id) = 3;
```

**Example input / output:**

```text
-- GIN Index Usage
SELECT * FROM posts WHERE tags @> '["A", "B", "C"]';

-- Normalized Join with Grouping
SELECT p.* FROM posts p
JOIN post_tags pt ON p.id = pt.post_id
WHERE tag_id IN ('A', 'B', 'C')
GROUP BY p.id HAVING count(DISTINCT pt.tag_id) = 3;
```

**Data:**

```sql
CREATE TABLE tags (id SERIAL PRIMARY KEY, name TEXT UNIQUE NOT NULL);
CREATE TABLE post_tags (post_id INTEGER REFERENCES posts(id), tag_id INTEGER REFERENCES tags(id));
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. GIN Index on Tags
6. Is schema simple?
7. Use GIN Index
8. Normalized Join with Grouping
9. Does the application needfull-text search?
10. Inverted-Index Table

