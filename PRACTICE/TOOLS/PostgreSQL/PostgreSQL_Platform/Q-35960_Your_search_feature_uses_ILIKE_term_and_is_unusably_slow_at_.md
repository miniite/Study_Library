---
type: "practice"
difficulty: "Deep"
---

# Your search feature uses ILIKE '%term%' and is unusably slow at scale; compare trigram (pg_trgm) indexes, full-text search, and moving to Elasticsearch and how you'd decide.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Full-text Search in Postgres](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Full-text_Search_in_Postgres.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with a search feature that uses ILIKE '%term%' in PostgreSQL and is experiencing performance issues at scale, we need to consider several options: trigram (pg_trgm) indexes, full-text search, and potentially moving to Elasticsearch. The goal is to optimize the query execution, reduce the load on the database server, and enhance the user experience.

Firstly, let's understand what ILIKE '%term%' does. It performs a case-insensitive pattern match against any substring in the column value. This approach can be slow because it requires scanning the entire table, which is inefficient for large datasets. To address this issue, we can explore different solutions:

Trigram Indexes (pg_trgm)
Full-Text Search
Moving to Elasticsearch

#### Triage
What are We Probing?

The interviewer is likely probing your understanding of:

The performance implications of using ILIKE '%term%'.
The trade-offs between trigram indexes, full-text search, and external storage solutions like Elasticsearch.
How to evaluate and choose the best solution based on specific requirements such as scale, query complexity, and performance needs.
Step-by-Step Solution From First Principles
1. Trigram Indexes (pg_trgm)

Trigram indexes can help by breaking words into trigrams (3-character sequences) and allowing pattern matching against these trigrams. This method is generally faster than ILIKE but still may not scale well for very large datasets.

#### Pros
No external dependencies.
Can be implemented using standard PostgreSQL extensions.

#### Cons
Requires additional storage space for the index.
May not handle complex search queries as effectively as full-text search or Elasticsearch.

To implement trigram indexes:

SQL
CREATE EXTENSION IF NOT EXISTS pg_trgm;

-- Create an index on the column you want to search
CREATE INDEX idx_trigrams ON your_table USING gin (your_column gist_trgm_ops);

-- Example query using trigram similarity
SELECT * FROM your_table WHERE your_column % 'term';
2. Full-Text Search

PostgreSQL's full-text search capabilities provide more powerful and flexible query options compared to ILIKE or even trigram indexes.

#### Pros
Supports advanced features like stemming, fuzzy matching, and relevance scoring.
Can handle complex queries with better performance than simple pattern matching.

#### Cons
More complex setup and maintenance.
Requires additional configuration and tuning.

To implement full-text search:

SQL
-- Add a tsvector column to the table if not already present
ALTER TABLE your_table ADD COLUMN if_not_exists your_column_tsvector TSVECTOR;

-- Populate the tsvector column using the TO_TSVECTOR function
UPDATE your_table SET your_column_tsvector = to_tsvector('simple', your_column);

-- Create an index on the tsvector column
CREATE INDEX idx_fts ON your_table USING GIN (your_column_tsvector);

-- Example query with full-text search
SELECT * FROM your_table WHERE to_tsquery('simple', 'term') @@ your_column_tsvector;
3. Moving to Elasticsearch

Elasticsearch is a powerful full-text search and analytics engine that can handle complex queries at scale.

#### Pros
Highly scalable.
Rich query capabilities including faceting, aggregations, and real-time indexing.
Good for handling large volumes of data.

#### Cons
Requires additional infrastructure and operational complexity.
Data duplication may be necessary to keep Elasticsearch synchronized with the main database.

To move to Elasticsearch:

Install and configure Elasticsearch.
Index your data in Elasticsearch.
Use a tool like Logstash or Kibana for real-time indexing if needed.
Decision Criteria

When deciding between these options, consider the following factors:

Query Complexity: Full-text search and Elasticsearch are better suited for complex queries involving stemming, fuzzy matching, and relevance scoring.
Performance Needs: Trigram indexes can be a good initial step for simpler cases but may not scale well. Elasticsearch is generally best for high-performance needs at scale.
Operational Complexity: Implementing full-text search in PostgreSQL or moving to Elasticsearch increases operational complexity but provides more robust features.
Cost and Resources: Assess the cost of adding new infrastructure versus optimizing existing queries.

#### Closing Summary

In summary, trigram indexes are a good starting point for simple pattern matching, while full-text search offers more advanced capabilities. Moving to Elasticsearch is recommended when dealing with complex queries at scale but requires additional resources and operational effort.

#### Common Mistakes/Follow-Ups
Overlooking the need for regular maintenance and tuning of full-text search configurations.
Ignoring potential data consistency issues when moving between PostgreSQL and Elasticsearch.
Not considering the impact on overall system performance during migration to Elasticsearch.

**Comparison:**

| Index Type   | Pros                                                                 | Cons                                                        |
|--------------|----------------------------------------------------------------------|-------------------------------------------------------------|
| Trigram      | Simple to implement, no external dependencies                         | Limited for complex queries                                 |
| Full-Text    | Advanced features (stemming, fuzzy matching)                         | Complex setup and maintenance                              |
| Elasticsearch | Highly scalable, rich query capabilities                            | Additional infrastructure and operational complexity        |

**Code:**

```sql
-- Trigram Index Example
CREATE EXTENSION IF NOT EXISTS pg_trgm;

CREATE INDEX idx_trigrams ON your_table USING gin (your_column gist_trgm_ops);

SELECT * FROM your_table WHERE your_column % 'term';

-- Full-Text Search Example
ALTER TABLE your_table ADD COLUMN if_not_exists your_column_tsvector TSVECTOR;

UPDATE your_table SET your_column_tsvector = to_tsvector('simple', your_column);

CREATE INDEX idx_fts ON your_table USING GIN (your_column_tsvector);

SELECT * FROM your_table WHERE to_tsquery('simple', 'term') @@ your_column_tsvector;
```

**Example input / output:**

```text
-- Initial Query
SELECT * FROM your_table WHERE title ILIKE '%term%';

-- Trigram Index Query
CREATE INDEX idx_trigrams ON your_table USING gin (title gist_trgm_ops);
EXPLAIN SELECT * FROM your_table WHERE title % 'term';

-- Full-Text Search Query
ALTER TABLE your_table ADD COLUMN if_not_exists title_tsvector TSVECTOR;
UPDATE your_table SET title_tsvector = to_tsvector('simple', title);

CREATE INDEX idx_fts ON your_table USING GIN (title_tsvector);
EXPLAIN SELECT * FROM your_table WHERE to_tsquery('simple', 'term') @@ title_tsvector;

-- Elasticsearch Integration Example
PUT /your_index/_doc/1
{
  "id": 1,
  "title": "Sample Title"
}

GET /your_index/_search
{
  "query": {
    "match": {"title": "term"}
  }
}
```

**Data:**

```json
{
  "table": {
    "name": "your_table",
    "columns": [
      {"name": "id", "type": "int"},
      {"name": "title", "type": "text"}
    ]
  }
}
```

**Flow:**

1. Initial ILIKE Query
2. Trigram Index
3. Full-Text Search
4. Elasticsearch Migration
5. Advanced Queries
6. Scalability and Performance

