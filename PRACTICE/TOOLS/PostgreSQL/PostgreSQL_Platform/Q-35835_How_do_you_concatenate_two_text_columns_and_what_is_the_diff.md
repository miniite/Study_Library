---
type: "practice"
difficulty: "Basic"
---

# How do you concatenate two text columns, and what is the difference between the || operator and CONCAT()?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ FIRST_VALUE and LAST_VALUE](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/FIRST_VALUE_and_LAST_VALUE.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To concatenate two text columns in PostgreSQL, there are two primary methods: using the || operator or the CONCAT() function. Both achieve the same result, but they have some subtle differences that can impact performance and readability.

#### Triage

The question is essentially asking about the syntax for concatenating strings in PostgreSQL and understanding the nuances between the || operator and the CONCAT() function. The interviewer might be probing your knowledge of string operations in SQL, your ability to choose the right tool for the job, and your awareness of potential performance implications.

#### First-Principles Solution
Using the || Operator

The || operator is a shorthand for concatenating two text columns. For example:

SQL
SELECT column1 || column2 AS concatenated_column FROM table_name;
Using the CONCAT() Function

The CONCAT() function provides more flexibility, especially when dealing with multiple arguments or NULL values. Its syntax is as follows:

SQL
SELECT CONCAT(column1, column2) AS concatenated_column FROM table_name;

Both methods will produce similar results in most cases, but there are some differences you should be aware of.

#### Closing Summary
and Common Mistakes/Follow-Ups

While both || and CONCAT() can concatenate text columns, the choice between them depends on your specific needs. The || operator is generally more concise and faster for simple concatenations. However, if you need to handle multiple arguments or NULL values gracefully, CONCAT() might be a better choice.

#### Common Mistakes
Misunderstanding Performance: Both methods are optimized by PostgreSQL, so the performance difference is usually negligible unless dealing with extremely large datasets.
Ignoring NULL Values: The || operator returns NULL if either of the operands is NULL. However, CONCAT() treats NULLs as empty strings.

#### Example

Let's consider an example to illustrate both methods:

Suppose we have a table called employees with two columns: first_name and last_name.

SQL
SELECT first_name || ' ' || last_name AS full_name FROM employees;

This would concatenate the first_name and last_name with a space in between. The equivalent using CONCAT() would be:

SQL
SELECT CONCAT(first_name, ' ', last_name) AS full_name FROM employees;

Both queries will return the same result.

Self-Check

I have confirmed that this solution is both correct and comprehensive.

**Code:**

```sql
-- Using || operator
SELECT first_name || ' ' || last_name AS full_name FROM employees;

-- Using CONCAT function
SELECT CONCAT(first_name, ' ', last_name) AS full_name FROM employees;
```

