---
type: "practice"
difficulty: "Basic"
---

# How would you write a SQL query to detect duplicate business keys that should be unique in a dimension table?

**Difficulty:** Basic · **Levels:** Fresher · [↑ Classic SQL Query Problems](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Classic_SQL_Query_Problems.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the problem of detecting duplicate business keys that should be uniquely identified in a dimension table, we need to ensure our SQL query can identify and highlight any instances where the same key appears more than once. This is crucial for maintaining data integrity and ensuring accurate reporting.

The core concept here involves using aggregate functions and common table expressions (CTEs) in SQL. We'll first group the records by the business key and then count occurrences of each key to flag duplicates.

**Code:**

```sql
WITH DuplicateKeys AS (
    SELECT
        business_key,
        COUNT(*) AS occurrence_count
    FROM
        dimension_table
    GROUP BY
        business_key
    HAVING
        COUNT(*) > 1
)
SELECT *
FROM DuplicateKeys;
```

**Example input / output:**

```

This query will return a result set containing the business_key and its occurrence count, where each row represents a unique business key that appears more than once in the dimension table.

Example output:

PLAINTEXT
Copy
| business_key | occurrence_count |
|--------------|------------------|
| BK0123       | 4                |
| BK4567       | 2                |
```

