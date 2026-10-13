---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd model a tags-per-article schema so filtering by multiple tags (AND semantics) stays index-friendly instead of degenerating into many self-joins.

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Activity Timeline Schema](<../../../../TOOLS/MySQL/MySQL_Platform/Activity_Timeline_Schema.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To efficiently handle tagging in an article database with AND semantics, we need to model our schema carefully to ensure that filtering by multiple tags remains index-friendly. The key challenge is to avoid degenerating into many self-joins, which can significantly degrade performance and increase complexity. We'll start from the basics of how a simple tagging system might look, identify the issues, and then present an optimized solution.

#### Triage
What's Being Probed

The interviewer is probing your understanding of:

Normalization vs Denormalization: How to balance between normalized tables for good data integrity and denormalized structures for better performance.
Indexing Strategy: Techniques to ensure efficient querying while minimizing the overhead of self-joins.
Query Optimization: Understanding how SQL queries can be crafted to leverage indexes effectively.

#### Assumptions
Each article can have multiple tags, but each tag should apply uniquely per article (no duplicate tag-article relationships).
We are primarily interested in articles that match all specified tags (AND semantics).

#### First-Principles Solution
Initial Schema

A straightforward initial schema might look like this:

SQL
CREATE TABLE Articles (
    id INT PRIMARY KEY,
    title VARCHAR(100)
);

CREATE TABLE Tags (
    id INT PRIMARY KEY,
    name VARCHAR(50) UNIQUE
);

CREATE TABLE ArticleTags (
    article_id INT,
    tag_id INT,
    FOREIGN KEY (article_id) REFERENCES Articles(id),
    FOREIGN KEY (tag_id) REFERENCES Tags(id)
);

This schema is normalized, meaning it avoids any redundancy but can suffer from performance issues when querying articles by multiple tags due to the need for self-joins.

Problem with Self-Joins

To filter articles that have all specified tags, a naive approach would be:

SQL
SELECT A.*
FROM Articles A
WHERE EXISTS (SELECT 1 FROM ArticleTags AT WHERE AT.article_id = A.id AND AT.tag_id IN (:tags))
GROUP BY A.id
HAVING COUNT(DISTINCT AT.tag_id) = :num_tags;

This query involves a correlated subquery and self-joins, which are slow for large datasets.

Optimized Solution: Using a Single Table with Tag Combinations

To keep the filtering index-friendly, we can denormalize the schema by combining tags into unique article-tag combinations. This approach avoids self-joins but increases the complexity of data maintenance.

SQL
CREATE TABLE ArticleTagCombinations (
    id INT PRIMARY KEY,
    article_id INT,
    tag_ids BIT(20) -- assuming up to 20 tags per article, each bit represents a tag's presence
);

#### Implementation
Details
Insert Logic: When inserting or updating an article with multiple tags, the tag_ids column is updated using bitwise operations.
Filtering Query:
SQL
SELECT A.*
FROM Articles A
WHERE (SELECT tag_ids FROM ArticleTagCombinations WHERE article_id = A.id) & :bit_mask = :bit_mask;

#### Example
Filtering by Tags 1, 3, and 5
tag_ids for an article with tags 1, 3, and 5 would be 0000000000000000010110.
Bitmask for tags 1, 3, and 5: 0000000000000000010110.

The query would then be:

SQL
SELECT A.*
FROM Articles A
WHERE (SELECT tag_ids FROM ArticleTagCombinations WHERE article_id = A.id) & 0b0000000000000000010110 = 0b0000000000000000010110;

#### Closing Summary
and Common Mistakes

This approach maintains a good balance between data integrity and performance. The main advantage is that the filtering query is straightforward and avoids self-joins, making it index-friendly.

#### Common Mistakes

Over-normalization: Failing to denormalize the schema when necessary.
Incorrect Bitwise Operations: Errors in setting or checking tag_ids during insertions and queries.
Ignoring Indexes: Not using appropriate indexes on columns like article_id and tag_ids.

#### Follow-Up
Questions
What if you need to support more than 20 tags per article?
How would this approach handle articles with a large number of tags?

**Comparison:**

```
FLOWCHART · MERMAID
erDiagram
    Articles ||--M{ ArticleTagCombinations : has many
    Tags ||--M{ ArticleTags : is related to
```

**Code:**

```

CREATE TABLE Articles ( id INT PRIMARY KEY, title VARCHAR(100) );

CREATE TABLE Tags ( id INT PRIMARY KEY, name VARCHAR(50) UNIQUE );

CREATE TABLE ArticleTags ( article_id INT, tag_id INT, FOREIGN KEY (article_id) REFERENCES Articles(id), FOREIGN KEY (tag_id) REFERENCES Tags(id) );

CODE
Copy

```sql
CREATE TABLE ArticleTagCombinations (
    id INT PRIMARY KEY,
    article_id INT,
    tag_ids BIT(20)
);
```

**Flow:**

1. has many
2. tag_ids
3. Article
4. ArticleTagCombinations
5. Tag

