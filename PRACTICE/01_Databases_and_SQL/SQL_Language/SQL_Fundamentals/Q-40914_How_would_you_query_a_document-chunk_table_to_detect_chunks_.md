---
type: "practice"
difficulty: "Intermediate"
---

# How would you query a document-chunk table to detect chunks whose token count far exceeds the embedding model's max input, which would have been silently truncated at ingest?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ RAG Metrics in SQL](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/RAG_Metrics_in_SQL.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this problem, we need to identify document chunks that, despite being larger than the maximum input size of an embedding model, were still processed and stored without error. This involves querying a table where each row represents a chunk with metadata such as its original document ID, token count, and any other relevant information.

#### Triaging the Question

The question is asking us to write a SQL query that identifies chunks whose token count exceeds the maximum allowable size for input into an embedding model. The challenge here is ensuring that these chunks are detected after they have been ingested, which means we need access to their original token counts or sizes, and possibly some indicator of whether they were processed without error.

#### Assumptions
We assume there is a table named document_chunks with columns like id, doc_id, token_count, and processed.
The max_input_size of the embedding model is known.
We want to flag chunks that exceed this limit by more than a certain threshold, say 20%.

#### First-Principles Solution

We need to compare each chunk's token count against the maximum allowed size. If the difference between the chunk’s token count and the max input size exceeds our threshold (e.g., 20%), we will flag it.

Here is how we can write the SQL query:

SQL
SELECT
    id,
    doc_id,
    token_count,
    processed,
    CASE
        WHEN token_count - max_input_size > 0.2 * max_input_size THEN 'Chunk exceeds limit'
        ELSE 'Within limits'
    END AS chunk_status
FROM
    document_chunks
WHERE
    token_count > max_input_size;

#### Closing Summary
and Common Mistakes/Follow-Ups

This query will return a list of chunks that exceed the threshold, indicating they were likely truncated during ingestion. The CASE statement checks if the token count is significantly higher than the maximum allowed size.

#### Common Mistakes
Incorrect Threshold Calculation: Forgetting to multiply the difference by 0.2 (or whatever the threshold percentage is).
Ignoring Ingestion Errors: Failing to ensure that chunks flagged are indeed those that were ingested and processed without error.
Missing Metadata Columns: Not including all necessary metadata columns in the query.
Additional Considerations
If there are multiple models with different max input sizes, you might need to parameterize this calculation.

#### Cons
ider adding a processed column to check if chunks have been handled properly before they exceed their limits.

**Comparison:**

| id | doc_id | token_count | processed | chunk_status        |
|----|--------|------------|----------|---------------------|
| 1  | 42     | 5000       | true     | Chunk exceeds limit |

**Code:**

```sql
SELECT
    id,
    doc_id,
    token_count,
    processed,
    CASE
        WHEN token_count - max_input_size > 0.2 * max_input_size THEN 'Chunk exceeds limit'
        ELSE 'Within limits'
    END AS chunk_status
FROM
    document_chunks
WHERE
    token_count > max_input_size;
```

**Example input / output:**

```

Given the sample data:

JSON
Copy
[
  {
    "id": 1,
    "doc_id": 42,
    "token_count": 5000,
    "max_input_size": 4096,
    "processed": true
  },
  {
    "id": 2,
    "doc_id": 43,
    "token_count": 4096,
    "max_input_size": 4096,
    "processed": true
  }
]

The query would output:

SQL
Copy
+----+-------+-------------+---------+---------------------+
| id | doc_id| token_count | processed| chunk_status        |
+----+-------+-------------+---------+---------------------+
| 1  | 42    | 5000        | true    | Chunk exceeds limit |
+----+-------+-------------+---------+---------------------+
```

**Data:**

```json
{
  "document_chunks": [
    {
      "id": 1,
      "doc_id": 42,
      "token_count": 5000,
      "max_input_size": 4096,
      "processed": true
    },
    {
      "id": 2,
      "doc_id": 43,
      "token_count": 4096,
      "max_input_size": 4096,
      "processed": true
    }
  ]
}
```

**Flow:**

1. No
2. Start
3. Is token_count >max_input_size?
4. C
5. Output flagged chunks
6. Output 'Within limits'
7. End

