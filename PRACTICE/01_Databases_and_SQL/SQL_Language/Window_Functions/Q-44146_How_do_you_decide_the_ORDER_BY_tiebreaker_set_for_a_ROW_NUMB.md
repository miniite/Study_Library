---
type: "practice"
difficulty: "Deep"
---

# How do you decide the ORDER BY tiebreaker set for a ROW_NUMBER-based dedup so the same survivor is chosen deterministically across environments and reruns?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Deterministic Deduplication](<../../../../01_Databases_and_SQL/SQL_Language/Window_Functions/Deterministic_Deduplication.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with window functions, particularly ROW_NUMBER(), to achieve deterministic deduplication, deciding on an appropriate ORDER BY tiebreaker is crucial. The goal is to ensure that the same row is always chosen in case of ties, across different environments and reruns. This involves understanding the data distribution, identifying potential tie scenarios, and selecting a stable column or combination of columns as the tiebreaker.

**Comparison:**

| id | value  | created_at           | rn |
|----|--------|---------------------|----|
| 103| banana | 2023-10-04T17:00:00Z | 1  |

**Code:**

```sql
-- Example query using ROW_NUMBER() with an ORDER BY tiebreaker
WITH ranked_data AS (
    SELECT
        id,
        value,
        created_at,
        -- Determine a good tiebreaker based on the nature of the data and use cases.
        ROW_NUMBER() OVER (PARTITION BY id
                           ORDER BY created_at DESC,
                                    value ASC) as rn
    FROM
        my_table
)
SELECT *
FROM ranked_data
WHERE rn = 1;
```

**Example input / output:**

```

Given the input data:

JSON
Copy
{
  "id": [101, 102, 103],
  "value": ["apple", "banana", "banana"],
  "created_at": ["2023-10-05T08:00:00Z", "2023-10-04T16:00:00Z", "2023-10-04T17:00:00Z"]
}

The output after applying the ROW_NUMBER() with a tiebreaker on created_at and then value would be:

SQL
Copy
id | value  | created_at           | rn
---|--------|---------------------|----
103| banana | 2023-10-04T17:00:00Z | 1

This ensures that the most recent and lexicographically smallest value is chosen in case of a tie.
```

**Data:**

```json
{
  "id": [101, 102, 103],
  "value": ["apple", "banana", "banana"],
  "created_at": ["2023-10-05T08:00:00Z", "2023-10-04T16:00:00Z", "2023-10-04T17:00:00Z"]
}
```

**Flow:**

1. No
2. Yes
3. Start
4. Is there a tie?
5. Return the first row
6. Choose based on ORDER BYcriteria
7. RN = 1, return

