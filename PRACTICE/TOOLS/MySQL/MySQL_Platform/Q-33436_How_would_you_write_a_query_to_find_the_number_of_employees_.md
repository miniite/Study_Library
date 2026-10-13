---
type: "practice"
difficulty: "Intermediate"
---

# How would you write a query to find the number of employees hired in each year?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Date Functions and Grouping](<../../../../TOOLS/MySQL/MySQL_Platform/Date_Functions_and_Grouping.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To find the number of employees hired in each year, we need to query an employee table that contains information about when each employee was hired. The key idea is to use a GROUP BY clause on the hire date and aggregate the count.

#### Triaging the Question

The question asks for a SQL query to retrieve the number of employees hired per year. This involves understanding the structure of the database, specifically an employees table that likely contains a column called hire_date. The interviewer is probing your ability to use SQL's aggregation functions and grouping mechanisms effectively.

#### First-Principles Solution
Identify the Table and Columns: Assume we have an employees table with columns such as employee_id, name, and hire_date.
Extract the Year from the Hire Date: Use a function to extract just the year part of the hire date.
Group by Year and Count Employees: Group the data by these years and count the number of employees hired in each year.
Writing the Query
SQL
SELECT
    YEAR(hire_date) AS hire_year,
    COUNT(*) AS num_employees_hired
FROM
    employees
GROUP BY
    YEAR(hire_date);

#### Closing Summary
+ Common Mistakes/Follow-Ups

This query provides a simple yet effective way to count the number of hires per year. Key points include:

Using YEAR() function to get just the year from the hire date.
Grouping by this extracted year and counting employees.

#### Common mistakes
might include not using the correct date function or incorrectly grouping, leading to incorrect results.

#### Example

If the employees table looks like:

PLAINTEXT
| employee_id | name  | hire_date       |
|-------------|-------|----------------|
| 1           | John  | 2015-03-01     |
| 2           | Mary  | 2016-07-15     |
| 3           | David | 2015-12-31     |

The query would return:

PLAINTEXT
| hire_year | num_employees_hired |
|-----------|--------------------|
| 2015      | 2                  |
| 2016      | 1                  |
Self-Check

This solution is correct and complete.

