---
type: "practice"
difficulty: "Intermediate"
---

# Since which version did MySQL support window functions, and how did people emulate them before?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Window Function Fundamentals and Performance](<../../../../01_Databases_and_SQL/SQL_Language/Window_Functions/Window_Function_Fundamentals_and_Performance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To answer this question, we'll break it down into several parts:

#### Overview
of Window Functions in SQL: We'll start by defining what window functions are and their importance.
Support Timeline in MySQL: We'll discuss the history of support for window functions in MySQL, identifying when they were first introduced.
Emulation Techniques Before Support: We’ll explore how developers worked around the lack of window functions using alternative methods.

Window functions allow us to perform calculations across a set of rows that are related to the current row. They can be incredibly useful for tasks like running totals, ranking records, and more.

MySQL 8.0 is when full support for window functions was introduced in 2017. Before this version, MySQL did not natively support these functions, requiring developers to use workarounds or other SQL flavors that did have the feature.

**Comparison:**

| user_id | num_pages | session_date | total_pages_read |
|---------|-----------|--------------|------------------|
| 1       | 50        | 2023-01-01   | 50               |
| 1       | 60        | 2023-01-02   | 110              |
| 2       | 70        | 2023-01-01   | 70               |
| 2       | 80        | 2023-01-03   | 150              |

**Code:**

```sql
-- Example of a window function in MySQL 8.0+
SELECT user_id,
       SUM(num_pages) OVER (PARTITION BY user_id ORDER BY session_date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) as total_pages_read
FROM reading_sessions;
```

**Example input / output:**

```text
-- SQL query using window functions (assuming MySQL version >= 8.0)
SELECT user_id,
       SUM(num_pages) OVER (PARTITION BY user_id ORDER BY session_date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) as total_pages_read
FROM reading_sessions;

-- Expected output:
+---------+----------------+
| user_id | total_pages_read|
+---------+----------------+
| 1       | 50              |
| 1       | 110             |
| 2       | 70              |
| 2       | 150             |
+---------+----------------+
```

**Data:**

```json
{
  "reading_sessions": [
    {"user_id": 1, "num_pages": 50, "session_date": "2023-01-01"},
    {"user_id": 1, "num_pages": 60, "session_date": "2023-01-02"},
    {"user_id": 2, "num_pages": 70, "session_date": "2023-01-01"},
    {"user_id": 2, "num_pages": 80, "session_date": "2023-01-03"}
  ]
}
```

**Flow:**

1. No
2. Yes
3. MySQL Version < 8.0
4. Has Window FunctionsSupport?
5. Use Subqueries, CommonTable Expressions (CTEs), orOther SQL Flavors
6. Utilize Window FunctionsDirectly in the Query

