---
type: "practice"
difficulty: "Basic"
---

# How would you write SQL to detect and count rows where a numeric amount is negative when the business says it should always be positive?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Data Validation Queries](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Data_Validation_Queries.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this SQL question, we need to identify and count rows in a table where a numeric column containing amounts is negative, but the business expects all these values to be non-negative. The key steps involve filtering out invalid records based on the specified criteria and counting them.

**Comparison:**

|id |amount|
|---|------|
| 1 | -50  |
| 2 | 100  |
| 3 | -75  |
| ...| ...  |

**Code:**

```sql
-- Example query to detect and count negative amounts

SELECT
    COUNT(*) AS negative_amount_count
FROM
    transactions
WHERE
    amount < 0;
```

**Example input / output:**

```text
Input:
transactions table with columns id, amount.

Output:
negative_amount_count = 123

Explanation: The query counts the number of rows where the 'amount' column has a value less than zero.
```

