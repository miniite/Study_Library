---
type: "practice"
difficulty: "Intermediate"
---

# How do NULL values behave in comparisons and aggregate functions in MySQL?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ MySQL Query Basics](<../../../../TOOLS/MySQL/MySQL_Platform/MySQL_Query_Basics.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

NULL values in MySQL can be tricky due to their special nature, which doesn’t follow the same rules as other data types. In this answer, I'll explain how NULLs behave in comparisons and aggregate functions from first principles, ensuring you have a solid grasp of these concepts.

#### Triaging the Question

The interviewer is probing your understanding of MySQL’s treatment of NULL values, particularly in comparison operations and aggregate functions like COUNT, SUM, etc. You need to understand that NULL is not just an absence of data but also introduces unique behavior into SQL queries.

#### First Principles Solution

NULLs behave uniquely because they represent the absence of a value. Here are key points:

Comparisons with NULL:

Comparing a NULL value with any other value, including another NULL, always results in UNKNOWN (indeterminate). This means that expressions like NULL = NULL, NULL <> NULL, or NULL < 5 all evaluate to UNKNOWN.

Aggregate Functions and NULLs:

Most aggregate functions handle NULL values gracefully but with some caveats.
For example, COUNT(*) counts the number of rows in a result set, including those with NULL values.
However, SUM(column_name) will not include NULL values in its calculations unless explicitly handled (e.g., using IFNULL() or COALESCE()).
Detailed Explanation
Comparisons with NULL
SQL
SELECT IFNULL(NULL = NULL, 'Result') AS result;  -- Result: NULL
SELECT IFNULL(NULL <> NULL, 'Result') AS result; -- Result: NULL

These comparisons return UNKNOWN, which is why using a function like IFNULL() or CASE statements can be useful to handle such cases.

Aggregate Functions and NULLs

COUNT(*): This counts all rows, including those with NULL values.

SQL
SELECT COUNT(*) FROM employees;  -- Counts all rows, including NULLs in other columns.

SUM(column_name): By default, it ignores NULLs unless explicitly handled.

SQL
SELECT SUM(salary) FROM salaries;  -- Ignores NULL values by default.

-- Handling NULLs explicitly:
SELECT SUM(COALESCE(salary, 0)) FROM salaries;  -- Sums all salaries, treating NULL as 0.

AVG(column_name): Similar to SUM, it ignores NULLs unless handled.

SQL
SELECT AVG(salary) FROM salaries;  -- Ignores NULL values by default.

-- Handling NULLs explicitly:
SELECT AVG(COALESCE(salary, 0)) FROM salaries;  -- Averages all non-NULL salaries.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, NULL values in MySQL are special and behave uniquely. Comparisons involving NULL always yield UNKNOWN, while aggregate functions like SUM and AVG ignore NULLs by default unless explicitly handled with functions like IFNULL() or COALESCE(). Understanding these nuances is crucial for writing robust SQL queries.

#### Common mistakes
include:

Forgetting that comparisons involving NULL return UNKNOWN.
Not handling NULL values properly in aggregate functions, leading to incorrect results.
Assuming NULL will be treated the same way as any other value without explicit handling.

#### Follow-ups
might explore how different database systems handle NULLs differently or discuss best practices for dealing with NULL values in your application logic.

**Code:**

```sql
-- Example of using COALESCE in a SUM function
SELECT SUM(COALESCE(salary, 0)) AS total_salary FROM salaries;
```

