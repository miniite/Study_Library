---
type: "practice"
difficulty: "Deep"
---

# How would you store per-request retrieval sets and compute Mean Reciprocal Rank of the gold chunk across queries in SQL for a retrieval-quality dashboard?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Vector Similarity in SQL](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Vector_Similarity_in_SQL.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this problem, we need to design an efficient database schema and write SQL queries that can handle per-request retrieval sets and compute Mean Reciprocal Rank (MRR) of gold chunks across multiple queries. The MRR is a performance metric used in information retrieval to evaluate the ranking quality of search results.

The key steps involve:

Storing request-response pairs.
Defining what constitutes a "gold chunk."
Calculating MRR for each query and then averaging it.

Let's break down the solution from first principles:

#### Triaging the Question

The interviewer is probing our understanding of SQL, database design, performance considerations, and how to compute a specific metric (MRR). Assumptions include:

A "gold chunk" is considered the correct or relevant item in the retrieval set.
The system handles multiple queries with varying request-response pairs.

#### First-Principles Solution

Database Schema:

We need tables for storing requests, responses, and gold chunks.

Calculating MRR:

For each query, determine the rank of the gold chunk in the retrieval set.
The reciprocal of this rank is taken, and then the average across all queries is calculated.

#### Performance Considerations

We need to ensure that the SQL queries are efficient, especially if dealing with large datasets.
Step-by-Step Solution
1. Database Schema Design

We'll have three main tables: Requests, Responses, and GoldChunks.

SQL
CREATE TABLE Requests (
    request_id INT PRIMARY KEY,
    query_text VARCHAR(255)
);

CREATE TABLE Responses (
    response_id INT PRIMARY KEY,
    request_id INT,
    retrieval_set JSON,
    FOREIGN KEY (request_id) REFERENCES Requests(request_id)
);

CREATE TABLE GoldChunks (
    chunk_id INT PRIMARY KEY,
    gold_chunk_text VARCHAR(255),
    request_id INT,
    FOREIGN KEY (request_id) REFERENCES Requests(request_id)
);
2. Calculating MRR

To compute the MRR, we need to:

Determine the rank of each gold chunk in its respective retrieval set.
Compute the reciprocal of this rank for each occurrence.
Average these reciprocals across all queries.

Here’s a SQL query that calculates MRR:

SQL
WITH Ranks AS (
    SELECT
        r.request_id,
        g.chunk_id,
        RANK() OVER (PARTITION BY r.request_id ORDER BY JSON_EXTRACT_SCALAR(r.retrieval_set, CONCAT('$[', pos, ']')) = gc.gold_chunk_text DESC) as rank
    FROM
        Requests r
    JOIN
        GoldChunks gc ON r.request_id = gc.request_id
    CROSS JOIN
        UNNEST(JSON_ARRAYAGG(JSON_NUMBER_VALUE(JSON_EXTRACT_SCALAR(r.retrieval_set, CONCAT('$[', pos, ']'))), 1)) WITH ORDINALITY t(pos, idx)
),
RanksWithMRR AS (
    SELECT
        request_id,
        rank
    FROM
        Ranks
    WHERE
        rank = 1 -- Only consider the top-ranked gold chunk per query
)
SELECT
    AVG(1.0 / rank) as MRR
FROM
    RanksWithMRR;

#### Closing Summary
and Common Mistakes/Follow-Ups
Ensure that the retrieval_set is properly structured in JSON to facilitate easy access.

#### Cons
ider indexing on relevant columns if dealing with large datasets for performance optimization.
The above query assumes that the gold chunk appears exactly once per retrieval set. If it can appear multiple times, additional logic will be needed.

**Code:**

```

CREATE TABLE Requests ( request_id INT PRIMARY KEY, query_text VARCHAR(255) );

CREATE TABLE Responses ( response_id INT PRIMARY KEY, request_id INT, retrieval_set JSON, FOREIGN KEY (request_id) REFERENCES Requests(request_id) );

CREATE TABLE GoldChunks ( chunk_id INT PRIMARY KEY, gold_chunk_text VARCHAR(255), request_id INT, FOREIGN KEY (request_id) REFERENCES Requests(request_id) );

CODE
Copy
```sql
WITH Ranks AS (
    SELECT
        r.request_id,
        g.chunk_id,
        RANK() OVER (PARTITION BY r.request_id ORDER BY JSON_EXTRACT_SCALAR(r.retrieval_set, CONCAT('$[', pos, ']')) = gc.gold_chunk_text DESC) as rank
    FROM
        Requests r
    JOIN
        GoldChunks gc ON r.request_id = gc.request_id
    CROSS JOIN
        UNNEST(JSON_ARRAYAGG(JSON_NUMBER_VALUE(JSON_EXTRACT_SCALAR(r.retrieval_set, CONCAT('$[', pos, ']'))), 1)) WITH ORDINALITY t(pos, idx)
),
RanksWithMRR AS (
    SELECT
        request_id,
        rank
    FROM
        Ranks
    WHERE
        rank = 1 -- Only consider the top-ranked gold chunk per query
)
SELECT
    AVG(1.0 / rank) as MRR
FROM
    RanksWithMRR;
```

**Flow:**

1. Requests
2. Responses
3. GoldChunks
4. Ranks
5. RanksWithMRR
6. MRR

