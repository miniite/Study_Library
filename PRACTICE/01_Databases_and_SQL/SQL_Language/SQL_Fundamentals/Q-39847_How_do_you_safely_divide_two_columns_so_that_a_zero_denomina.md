---
type: "practice"
difficulty: "Intermediate"
---

# How do you safely divide two columns so that a zero denominator returns NULL instead of an error?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ NULL Handling](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/NULL_Handling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the problem of safely dividing one column by another in SQL, especially when the divisor could be zero, we need to return NULL without causing an error. This can be achieved using conditional logic and control structures provided by SQL.

TL;DR:

Use a CASE statement or the COALESCE function to handle division by zero gracefully, ensuring that any attempt to divide by zero results in NULL instead of an error.

#### Triaging the Question

The core of this question is about handling potential errors during arithmetic operations. Specifically, when dividing one column by another, we want to avoid an error if the denominator (the divisor) is 0. We need a way to return NULL in such cases.

#### Assumptions
and Context
The columns involved are numeric.
We assume that division by zero should result in NULL.
This solution needs to be robust enough to handle edge cases without causing SQL errors.

#### First Principles Solution
Using CASE Statement

The CASE statement is a powerful construct for conditional logic. It allows us to perform different actions based on certain conditions. Here’s how we can use it:

SQL
SELECT
    column1,
    column2,
    CASE
        WHEN column2 = 0 THEN NULL
        ELSE column1 / column2
    END AS safe_division
FROM your_table;
Using COALESCE Function

The COALESCE function returns the first non-null value in a list. We can use it to replace the result of division with NULL if the denominator is zero:

SQL
SELECT
    column1,
    column2,
    COALESCE(column1 / column2, NULL) AS safe_division
FROM your_table;
Using IF Function (MySQL-specific)

In MySQL and some other databases that support it, you can also use the IF function to handle this:

SQL
SELECT
    column1,
    column2,
    IF(column2 = 0, NULL, column1 / column2) AS safe_division
FROM your_table;

#### Closing Summary
and Common Mistakes/Follow-ups

In summary, we can safely divide two columns by using conditional logic. The CASE statement or COALESCE function are effective methods to avoid division by zero errors, ensuring that the result is NULL in such cases.

#### Common Mistakes
Forgetting the Edge Case: Not handling the situation where the denominator could be zero.
Incorrect Use of Functions: Using incorrect SQL functions (e.g., trying to use arithmetic operations directly instead of conditional logic).
Ignoring Database-Specific Syntax: Different databases have different syntax for conditionals. For example, COALESCE is not available in all SQL dialects.

#### Follow-Ups
How would you handle this in a more complex query with multiple conditions?
What performance considerations should be taken into account if this operation needs to be performed frequently?

**Comparison:**

| column1 | column2 | safe_division |
|---------|---------|---------------|
| 10      | 5       | 2.0           |
| 15      | 3       | 5.0           |
| 20      | 0       | NULL          |

**Code:**

```sql
-- CASE Statement Example
SELECT
    column1,
    column2,
    CASE
        WHEN column2 = 0 THEN NULL
        ELSE column1 / column2
    END AS safe_division
FROM your_table;

-- COALESCE Function Example
SELECT
    column1,
    column2,
    COALESCE(column1 / column2, NULL) AS safe_division
FROM your_table;

-- IF Function (MySQL-specific)
SELECT
    column1,
    column2,
    IF(column2 = 0, NULL, column1 / column2) AS safe_division
FROM your_table;
```

**Example input / output:**

```text
Input:
- Table 'your_table' with columns ['column1', 'column2']
  Rows:
  - (10, 5)
  - (15, 3)
  - (20, 0)

Output:
- column1 | column2 | safe_division
- 10      | 5       | 2.0
- 15      | 3       | 5.0
- 20      | 0       | NULL
```

**Data:**

```json
{
  "your_table": {
    "columns": ["column1", "column2"],
    "rows": [
      {"column1": 10, "column2": 5},
      {"column1": 15, "column2": 3},
      {"column1": 20, "column2": 0}
    ]
  }
}
```

**Flow:**

1. Yes
2. No
3. Input
4. Is Denominator Zero?
5. Return NULL
6. Perform Division
7. Output Safe Result

