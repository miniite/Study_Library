---
type: "practice"
difficulty: "Basic"
---

# How do you use the ORDER BY clause to sort results in ascending and descending order?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ T-SQL Basics](<../../../../TOOLS/SQL_Server/T-SQL/T-SQL_Basics.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To sort query results using the ORDER BY clause in T-SQL, you'll typically specify a column or expression that you want to base your sorting on, along with an optional direction (ascending or descending). Here's a brief overview and then we'll delve into the details.

The ORDER BY clause is used to return data from a table or view sorted in ascending (A-Z) or descending (Z-A) order. It can be applied to one or more columns and you can also use expressions, such as functions or calculations, for sorting.

Now let's break down how to use the ORDER BY clause:

Basic Syntax:

SQL
SELECT column_name(s)
FROM table_name
ORDER BY column_name [ASC | DESC];

Sorting by One Column Ascending: For example, if you have a table named Employees with an Age column, and you want to sort the results in ascending order (youngest to oldest), your query would look like this:

SQL
SELECT Name, Age
FROM Employees
ORDER BY Age ASC;

Sorting by One Column Descending: If you want to sort the same Employees table by age in descending order (oldest to youngest):

SQL
SELECT Name, Age
FROM Employees
ORDER BY Age DESC;

Multiple Columns Sorting: You can also sort by multiple columns. For instance, if you want to first sort by Department and then within each department by Age, your query would be:

SQL
SELECT Name, Department, Age
FROM Employees
ORDER BY Department ASC, Age DESC;

Using Functions with ORDER BY: You can use functions as well to perform sorting, for example, if you want to sort by the length of an employee's name in descending order:

SQL
SELECT Name, Age
FROM Employees
ORDER BY LEN(Name) DESC;

Let's summarize: The ORDER BY clause is fundamental for organizing query results. By default, it sorts in ascending order (ASC), but you can specify the direction explicitly with ASC or DESC. You can sort by one or multiple columns and even use functions to create custom sorting criteria.

#### Common mistakes
include forgetting to add an ORDER BY clause when sorting results, or misunderstanding the difference between ASC and DESC. Always double-check your query to ensure it performs as intended.

