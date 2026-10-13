---
type: "practice"
difficulty: "Basic"
---

# Write a query using UPPER and LOWER to standardize the case of a name column.

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ String Functions](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Functions/String_Functions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To standardize the case of names in a database, we can use SQL functions UPPER and LOWER. The core idea is to convert all names to either uppercase or lowercase so that data quality and consistency are maintained. For this basic SQL task, I'll show how to write an SQL query using these functions.

#### Triaging the Question

The interviewer is likely looking for your understanding of string manipulation in SQL and the ability to apply simple functions like UPPER and LOWER. The key assumptions here are that you know what a column named "name" contains, and we're working with a typical database schema.

Step-by-Step Solution from First Principles
Identify the Table and Column: Assume we have a table called employees with a column name.
Apply the Function: Use the UPPER or LOWER function to convert the name column.
Select All Relevant Data: Typically, you would select all rows unless there are specific conditions.
Full Solution

Here's an example SQL query that converts the name column to uppercase:

SQL
SELECT UPPER(name) AS standardized_name
FROM employees;

This query selects each row from the employees table and applies the UPPER function to the name column, returning a new column named standardized_name.

#### Closing Summary
and Common Mistakes/Follow-Ups

In this simple task:

Make sure you understand which case (uppercase or lowercase) is required.
Be aware that applying UPPER or LOWER might affect sorting and indexing in more complex queries.

A common mistake could be forgetting to alias the output column, leading to an ambiguous query. Always use meaningful aliases for clarity.

Closing

This approach ensures consistency across your database by normalizing name data to a standard case format. If you need to ensure both uppercase and lowercase cases are handled correctly, consider applying both functions in different scenarios or using a more sophisticated method like case conversion in multi-step queries.

**Code:**

```sql
SELECT UPPER(name) AS standardized_name
FROM employees;
```

