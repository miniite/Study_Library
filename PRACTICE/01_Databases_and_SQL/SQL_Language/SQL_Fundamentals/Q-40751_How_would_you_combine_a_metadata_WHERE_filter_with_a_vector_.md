---
type: "practice"
difficulty: "Intermediate"
---

# How would you combine a metadata WHERE filter with a vector similarity ORDER BY in a single pgvector query, and why is the filter useful for RAG?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Vector Similarity in SQL](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Vector_Similarity_in_SQL.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this SQL query challenge, we'll focus on combining a WHERE clause to apply metadata filters with an ORDER BY clause based on vector similarity in PostgreSQL using the pgvector extension. The WHERE filter is crucial for RAG (Retrieval-Augmented Generation) as it helps narrow down relevant documents before scoring them by vector similarity, thus improving efficiency and relevance.

The core idea here is to leverage PostgreSQL's powerful query capabilities to combine metadata-based filtering with content-based ordering based on vector embeddings. We'll break this down step-by-step from first principles:

Understand the Problem: We need a SQL query that filters documents using metadata (e.g., tags, category) and then orders the results by how similar their vector representations are to a given query.

Set Up the Environment: Ensure pgvector is installed and configured in your PostgreSQL database.

Define Tables and Data: Create tables for storing content and associated metadata, along with embedding vectors.

Write the Query: Construct an SQL statement that combines both filtering and ordering based on vector similarity.

#### Triage
What’s Being Probed
The interviewer is probing your understanding of:
pgvector functionality and integration with PostgreSQL.
SQL query construction and optimization techniques.
Metadata usage in database queries for refining search results.
Vector similarity calculations and their application in RAG systems.

#### Assumptions
and Reasoning from First Principles
We have a table named documents that stores content along with metadata like category, tags, etc., as well as vector embeddings computed using some natural language processing technique.
The user input is a query represented by a vector embedding which we'll call query_vector.
Our goal is to first filter documents based on metadata and then order the results by their similarity to query_vector.
SQL Query Construction

Here’s how you can construct such a query:

SQL
-- Example Query
SELECT *
FROM documents
WHERE category = 'Tech' AND tags @> ARRAY['Python', 'AI']
ORDER BY similarity(embedding, query_vector) DESC
LIMIT 10;
Metadata Filter (WHERE Clause): We use the category and tags columns to filter documents. The @> operator checks if the array of tags contains all specified elements.
Vector Similarity Order: The ORDER BY similarity(embedding, query_vector) DESC clause orders results based on how similar their vector embedding is to the query vector.

#### Closing Summary

This query effectively combines metadata filters with vector similarity sorting. By filtering documents first, we reduce the number of vectors that need to be compared, making the query more efficient and faster. This is particularly useful in RAG systems where relevance must be high but speed also matters.

#### Common Mistakes/Follow-Ups
Mistake 1: Not using the @> operator for metadata filtering can lead to incorrect results or poor performance.
Mistake 2: Failing to use the LIMIT clause can result in slow queries if too many documents are returned before ordering.

#### Follow-Up
Discuss edge cases like no matching vectors, multiple filters and their precedence, and how vector similarity is calculated.

**Comparison:**

| id | title                  | category   | tags                   | embedding               |
|----|------------------------|-----------|------------------------|-------------------------|
| 1  | Introduction to AI     | Tech      | [AI, Machine Learning] | [0.2, -0.3, 0.4]        |
| 2  | Python Programming Guide | Tech   | [Python, Programming]  | [-0.1, 0.5, 0.6]        |

**Code:**

```sql
-- Example Query
SELECT *
FROM documents
WHERE category = 'Tech' AND tags @> ARRAY['Python', 'AI']
ORDER BY similarity(embedding, query_vector) DESC
LIMIT 10;
```

**Example input / output:**

```text
-- Example Input -> Output
Input:
SELECT *
FROM documents
WHERE category = 'Tech' AND tags @> ARRAY['Python', 'AI']
ORDER BY similarity(embedding, query_vector) DESC
LIMIT 10;

Output (Sample):
[
  {
    "id": 2,
    "title": "Python Programming Guide",
    "category": "Tech",
    "tags": ["Python", "Programming"],
    "embedding": [-0.1, 0.5, 0.6]
  },
  {
    "id": 1,
    "title": "Introduction to AI",
    "category": "Tech",
    "tags": ["AI", "Machine Learning"],
    "embedding": [0.2, -0.3, 0.4]
  }
]
```

**Data:**

```json
{
  "documents": [
    {
      "id": 1,
      "title": "Introduction to AI",
      "category": "Tech",
      "tags": ["AI", "Machine Learning"],
      "embedding": [0.2, -0.3, 0.4]
    },
    {
      "id": 2,
      "title": "Python Programming Guide",
      "category": "Tech",
      "tags": ["Python", "Programming"],
      "embedding": [-0.1, 0.5, 0.6]
    }
  ],
  "query_vector": [0.1, -0.3, 0.4]
}
```

**Flow:**

1. Yes
2. No
3. Start Query
4. Is Metadata Filter Needed?
5. Apply WHERE clause withmetadata filters
6. Compute Vector Similarity forRemaining Documents
7. Sort by similarity and limitresults
8. Return Results
9. End Query

