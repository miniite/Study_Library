---
type: "practice"
difficulty: "Deep"
---

# How do you compute a de-duplicated 'latest version per key' from a CDC stream where ties on the ordering timestamp are possible?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Deterministic Deduplication](<../../../../01_Databases_and_SQL/SQL_Language/Window_Functions/Deterministic_Deduplication.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the problem of computing a de-duplicated latest version per key from a Change Data Capture (CDC) stream, especially when there can be ties on the ordering timestamp, we need to leverage window functions in SQL. The solution involves using Common Table Expressions (CTEs) and window functions like ROW_NUMBER() or RANK(). This approach ensures that only the latest version of each key is retained while handling potential ties in timestamps gracefully.

**Comparison:**

| id | version | timestamp          |
|----|---------|--------------------|
| 001| 2       | 2023-10-01T15:00:00Z|
| 001| 1       | 2023-10-01T14:59:59Z|
| 002| 3       | 2023-10-01T16:00:00Z|
| 002| 2       | 2023-10-01T15:59:59Z|

**Code:**

```sql
WITH DedupedData AS (
    SELECT
        id,
        version,
        timestamp,
        ROW_NUMBER() OVER (PARTITION BY id ORDER BY timestamp DESC) as rn
    FROM
        cdc_stream
)
SELECT
    id,
    version,
    timestamp
FROM
    DedupedData
WHERE
    rn = 1;
```

**Example input / output:**

```text
Input:
{
  "cdc_stream": [
    { "id": "001", "version": 2, "timestamp": "2023-10-01T15:00:00Z" },
    { "id": "001", "version": 1, "timestamp": "2023-10-01T14:59:59Z" },
    { "id": "002", "version": 3, "timestamp": "2023-10-01T16:00:00Z" },
    { "id": "002", "version": 2, "timestamp": "2023-10-01T15:59:59Z" }
  ]
}

Output:
[
  { "id": "001", "version": 2, "timestamp": "2023-10-01T15:00:00Z" },
  { "id": "002", "version": 3, "timestamp": "2023-10-01T16:00:00Z" }
]
```

**Data:**

```json
{
  "cdc_stream": [
    { "id": "001", "version": 2, "timestamp": "2023-10-01T15:00:00Z" },
    { "id": "001", "version": 1, "timestamp": "2023-10-01T14:59:59Z" },
    { "id": "002", "version": 3, "timestamp": "2023-10-01T16:00:00Z" },
    { "id": "002", "version": 2, "timestamp": "2023-10-01T15:59:59Z" }
  ]
}
```

**Flow:**

1. Yes
2. No
3. No
4. Input CDC Stream
5. Does the stream have ties ontimestamp?
6. Use ROW_NUMBER() orRANK()
7. Are there duplicate IDs withsame timestamp?
8. Select latest version per key
9. Handle ties using atie-breaking column likesequence number
10. Output deduped data

