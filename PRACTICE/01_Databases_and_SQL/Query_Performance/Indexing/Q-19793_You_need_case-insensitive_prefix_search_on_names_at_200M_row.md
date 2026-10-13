---
type: "practice"
difficulty: "Deep"
---

# You need case-insensitive prefix search on names at 200M rows. Compare a functional index on lower(name) with text_pattern_ops, a trigram index, and a dedicated search service.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Functional Indexes](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Functional_Indexes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the requirement of case-insensitive prefix search on names at 200M rows, we need to compare three different approaches: using a functional index with lower(name), utilizing a trigram index (text_pattern_ops), and employing a dedicated search service. Each method has its own trade-offs in terms of complexity, performance, and scalability.

#### Triage

The core question is to determine which approach provides the best balance between speed, efficiency, and maintainability for case-insensitive prefix searches on large datasets. We'll consider the functional index using lower(name), a trigram index (text_pattern_ops), and a dedicated search service like Elasticsearch or Amazon DynamoDB.

#### Assumptions
The database is PostgreSQL.
The table contains 200 million rows of name data.
The requirement is to support case-insensitive prefix searches efficiently.

#### First Principles Solution
Functional Index with lower(name)

A functional index on lower(name) can be created by storing the lowercased version of the names and indexing them. This approach works well for simple, exact match queries but may not be as efficient for prefix searches due to the nature of case-insensitivity.

Create a Functional Index

SQL
CREATE INDEX idx_name_lower ON table_name (lower(name));

Query Performance

For full-text matches: Efficient.
For prefix searches: Less efficient because it requires checking a broader range of rows.
Trigram Index (text_pattern_ops)

A trigram index can be used to support fuzzy string matching, which is effective for prefix and infix searches but may not be the best choice for case-insensitive operations directly. It involves splitting names into 3-character sequences (trigrams) and creating an index on these sequences.

Create a Trigram Index

SQL
CREATE EXTENSION IF NOT EXISTS pg_trgm;
CREATE INDEX idx_name_trigram ON table_name USING gist (name gin_trgm_ops);

Query Performance

Full-text matching: Good.
Prefix searches: Fair, but not optimal due to trigram nature.
Dedicated Search Service

A dedicated search service like Elasticsearch or Amazon DynamoDB can provide more advanced text processing capabilities, including case-insensitive prefix searches, with better performance and scalability for large datasets.

Setup and Mapping

JSON
PUT /names_index
{
  "settings": {
    "analysis": {
      "filter": {
        "lowercase_filter": { "type": "lowercase" }
      },
      "analyzer": {
        "name_analyzer": {
          "tokenizer": "standard",
          "filter": ["lowercase_filter"]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "name": { "type": "text", "analyzer": "name_analyzer" }
    }
  }
}

Search Query

JSON
GET /names_index/_search
{
  "query": {
    "prefix": {
      "name": "john"
    }
  }
}

#### Closing Summary
and Common Mistakes/Follow-Ups
Functional Index with lower(name): While straightforward, it may not be the most efficient for prefix searches.
Trigram Index (text_pattern_ops): Provides good fuzzy matching but is not optimized for case-insensitive prefix searches.
Dedicated Search Service: Offers better performance and scalability for complex queries like case-insensitive prefix searches.

In practice, a dedicated search service would likely provide the best balance between speed, efficiency, and maintainability. However, for smaller datasets or simpler requirements, a functional index with lower(name) might be sufficient.

#### Common Mistakes
Overlooking the performance implications of case-insensitive operations.
Not considering the trade-offs between simplicity and efficiency.
Failing to evaluate the scalability and maintenance overhead of different solutions.
Final Thoughts

For 200 million rows, a dedicated search service like Elasticsearch would likely offer the best performance and flexibility. However, for smaller or simpler use cases, a functional index with lower(name) might be adequate.

