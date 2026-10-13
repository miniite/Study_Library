---
type: "practice"
difficulty: "Basic"
---

# How do you extract the year, month, and day from a date column?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Date Functions and Grouping](<../../../../TOOLS/MySQL/MySQL_Platform/Date_Functions_and_Grouping.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To extract the year, month, and day from a date column in MySQL, you use built-in functions like YEAR(), MONTH(), and DAY() to retrieve the respective components of the date. This is a fundamental operation when working with dates in SQL databases.

#### Triaging the Question

The question essentially asks for how to decompose a date into its constituent parts (year, month, day). The interviewer might be probing your familiarity with MySQL functions and basic date handling capabilities.

#### First-Principles Solution

In MySQL, you can extract the year, month, and day from a DATE or DATETIME column using the following SQL functions:

SQL
-- Assuming there is a table named 'employees' with a date column 'hire_date'
SELECT
    YEAR(hire_date) AS year,
    MONTH(hire_date) AS month,
    DAY(hire_date) AS day
FROM
    employees;

#### Closing Summary
and Common Mistakes/Follow-Ups

This query will return the year, month, and day for each record in the employees table. Here are some common mistakes to avoid:

Ensure that you use the correct date function: YEAR(), MONTH(), and DAY() expect a valid DATE or DATETIME value.
If your column is not named 'hire_date', make sure to replace it with the actual column name.

#### Follow-Ups
Handling NULL Values: What if some records have no hire date?
Date Range Filtering: How would you filter employees hired in a specific year or month?

**Code:**

```sql
SELECT
    YEAR(hire_date) AS year,
    MONTH(hire_date) AS month,
    DAY(hire_date) AS day
FROM
    employees;
```

