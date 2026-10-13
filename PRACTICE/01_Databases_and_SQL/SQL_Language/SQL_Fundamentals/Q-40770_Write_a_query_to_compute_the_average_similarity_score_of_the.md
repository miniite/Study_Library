---
type: "practice"
difficulty: "Intermediate"
---

# Write a query to compute the average similarity score of the top-retrieved chunk per query to spot low-confidence retrievals.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ RAG Metrics in SQL](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/RAG_Metrics_in_SQL.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this problem, we need to write an SQL query that calculates the average similarity score of the top-retrieved chunks per query. The goal is to identify any queries where the retrieval confidence might be low. Let's break down the steps required to achieve this.

#### Triaging the Question

The question asks for a query to compute the average similarity score of the top-retrieved chunk per query. This involves two main components:

Retrieval: Identifying the top-retrieved chunk per query.
Similarity Score Calculation: Calculating the average similarity score of these chunks.

#### Assumptions
We have a table retrievals that contains the retrieved chunks along with their similarity scores and associated queries.
The table structure might look something like this:
SQL
CREATE TABLE retrievals (
  id INT PRIMARY KEY,
  query_id INT NOT NULL,
  chunk TEXT NOT NULL,
  similarity_score FLOAT NOT NULL
);
Each query_id can have multiple rows, each representing a different chunk retrieved for that query.
We assume there is a threshold or criterion to determine which chunk is the top-retrieved.

#### First-Principles Solution
Identify Top-Retrieved Chunk: For each query_id, identify the top-retrieved chunk based on the highest similarity score.
Calculate Average Similarity Score: Compute the average of these top-similarity scores per query.

Let's start by identifying the top-retrieved chunk for each query using a window function to rank the chunks and then selecting the top one:

SQL
WITH ranked_chunks AS (
  SELECT
    query_id,
    chunk,
    similarity_score,
    ROW_NUMBER() OVER (PARTITION BY query_id ORDER BY similarity_score DESC) as ranking
  FROM
    retrievals
)
SELECT
  query_id,
  MAX(similarity_score) as max_similarity_score
FROM
  ranked_chunks
WHERE
  ranking = 1
GROUP BY
  query_id;

This common table expression (CTE) uses the ROW_NUMBER() function to rank chunks within each query_id by their similarity score. The outer query then filters for the top-ranked chunk and groups by query_id.

Calculate Average Similarity Score: To calculate the average, we need to join this result with the original table to get the scores and then compute the average.
SQL
WITH ranked_chunks AS (
  SELECT
    query_id,
    chunk,
    similarity_score,
    ROW_NUMBER() OVER (PARTITION BY query_id ORDER BY similarity_score DESC) as ranking
  FROM
    retrievals
),
top_retrieval_scores AS (
  SELECT
    rc.query_id,
    MAX(rc.similarity_score) as max_similarity_score
  FROM
    ranked_chunks rc
  WHERE
    rc.ranking = 1
  GROUP BY
    query_id
)
SELECT
  top_retrieval_scores.query_id,
  AVG(retrievals.similarity_score) as avg_similarity_score
FROM
  top_retrieval_scores
JOIN
  retrievals ON top_retrieval_scores.query_id = retrievals.query_id AND top_retrieval_scores.max_similarity_score = retrievals.similarity_score
GROUP BY
  top_retrieval_scores.query_id;

This query first identifies the top-retrieved chunks and then joins back to get all the relevant similarity scores for those chunks, finally calculating their average.

#### Closing Summary
+ Common Mistakes/Follow-Ups

The final SQL solution provides a way to compute the average similarity score of the top-retrieved chunk per query. This is useful in identifying low-confidence retrievals by looking at queries where the average similarity score is notably lower than expected. Common mistakes include not properly ranking the chunks, incorrectly joining tables, or missing steps in aggregation.

For follow-ups, you might consider optimizing performance if dealing with large datasets (e.g., using indexes, partitioning), or adding additional filtering criteria to refine the results further.

**Comparison:**

```
Query ID
Is Top-Chunk?
Similarity Score
Retrieval Table
Ranking
Average Score
```

**Code:**

```sql
WITH ranked_chunks AS (
  SELECT
    query_id,
    chunk,
    similarity_score,
    ROW_NUMBER() OVER (PARTITION BY query_id ORDER BY similarity_score DESC) as ranking
  FROM
    retrievals
),
top_retrieval_scores AS (
  SELECT
    rc.query_id,
    MAX(rc.similarity_score) as max_similarity_score
  FROM
    ranked_chunks rc
  WHERE
    rc.ranking = 1
  GROUP BY
    query_id
)
SELECT
  top_retrieval_scores.query_id,
  AVG(retrievals.similarity_score) as avg_similarity_score
FROM
  top_retrieval_scores
JOIN
  retrievals ON top_retrieval_scores.query_id = retrievals.query_id AND top_retrieval_scores.max_similarity_score = retrievals.similarity_score
GROUP BY
  top_retrieval_scores.query_id;
```

**Example input / output:**

```text
WITH ranked_chunks AS (
  SELECT
    query_id,
    chunk,
    similarity_score,
    ROW_NUMBER() OVER (PARTITION BY query_id ORDER BY similarity_score DESC) as ranking
  FROM
    retrievals
),
top_retrieval_scores AS (
  SELECT
    rc.query_id,
    MAX(rc.similarity_score) as max_similarity_score
  FROM
    ranked_chunks rc
  WHERE
    rc.ranking = 1
  GROUP BY
    query_id
)
SELECT
  top_retrieval_scores.query_id,
  AVG(retrievals.similarity_score) as avg_similarity_score
FROM
  top_retrieval_scores
JOIN
  retrievals ON top_retrieval_scores.query_id = retrievals.query_id AND top_retrieval_scores.max_similarity_score = retrievals.similarity_score
GROUP BY
  top_retrieval_scores.query_id;

-- Expected Output:
+------------+----------------+
| query_id   | avg_similarity_score |
+------------+----------------+
| 101        | 0.90              |
| 102        | 0.775             |
+------------+----------------+
```

**Data:**

```sql
CREATE TABLE retrievals (
  id INT PRIMARY KEY,
  query_id INT NOT NULL,
  chunk TEXT NOT NULL,
  similarity_score FLOAT NOT NULL
);
INSERT INTO retrievals (id, query_id, chunk, similarity_score) VALUES
(1, 101, 'Chunk A', 0.85),
(2, 101, 'Chunk B', 0.90),
(3, 102, 'Chunk C', 0.75),
(4, 102, 'Chunk D', 0.80);
```

**Flow:**

1. Identify Top-Chunks
2. Filter by Ranking
3. Top-Similarity Scores
4. Avg Similarity Score
5. Output Average Scores

