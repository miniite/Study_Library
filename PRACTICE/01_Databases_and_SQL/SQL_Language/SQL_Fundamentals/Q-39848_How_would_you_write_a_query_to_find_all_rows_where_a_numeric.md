---
type: "practice"
difficulty: "Basic"
---

# How would you write a query to find all rows where a numeric column falls between two values, inclusive?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Data Validation Queries](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Data_Validation_Queries.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To write an SQL query that finds all rows in a table where a specific numeric column's value falls within a given range (inclusive), you need to use the BETWEEN operator or >= and <=. This is a fundamental concept for querying data based on numerical ranges.

#### Triaging the Question

The question is asking how to write an SQL query that selects rows where the values of a numeric column fall between two specified numbers, inclusive. The interviewer might be probing your understanding of SQL syntax and logical operators. We'll assume you're working with a table named employees and a numeric column called salary.

#### First-Principles Solution

The basic SQL query to solve this problem would use the BETWEEN operator or a combination of >= and <=. Here’s how each approach works:

Using BETWEEN Operator
SQL
SELECT * FROM employees WHERE salary BETWEEN lower_bound AND upper_bound;
Using >= and <=
SQL
SELECT * FROM employees WHERE salary >= lower_bound AND salary <= upper_bound;

In both cases, you need to replace lower_bound and upper_bound with the actual numeric values or variables representing your range. For example:

SQL
-- Example query using BETWEEN operator
SELECT * FROM employees WHERE salary BETWEEN 50000 AND 100000;

-- Example query using >= and <=
SELECT * FROM employees WHERE salary >= 50000 AND salary <= 100000;

#### Closing Summary
+ Common Mistakes/Follow-Ups

The key is to ensure that the boundaries are inclusive, which the BETWEEN operator handles automatically. However, you should be aware of potential pitfalls such as using incorrect data types or not handling null values properly.

If asked a follow-up, you might discuss performance considerations (e.g., indexing on the column), edge cases like handling very large ranges, and how to handle NULL values in the salary column.

**Code:**

```sql
-- Example query using BETWEEN operator
SELECT * FROM employees WHERE salary BETWEEN 50000 AND 100000;

-- Example query using >= and <=
SELECT * FROM employees WHERE salary >= 50000 AND salary <= 100000;
```

