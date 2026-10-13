---
type: "practice"
difficulty: "Basic"
---

# How do you sort query results and what is the default sort direction of ORDER BY?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Ordering and Limits](<../../../../TOOLS/MySQL/MySQL_Platform/Ordering_and_Limits.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To start, let's quickly cover how to sort query results in MySQL and understand its default behavior.

TL;DR:

In MySQL, sorting query results can be done using the ORDER BY clause. By default, if no direction is specified, it sorts in ascending order (ASC). You can explicitly specify ASC or DESC for ascending or descending order respectively.

#### Triaging the Question

The core of this question revolves around understanding the basics of the ORDER BY clause and its default behavior. The interviewer might be probing your knowledge on SQL syntax and database operations.

#### First-Principles Solution
Understanding the ORDER BY Clause

In a MySQL query, you can use the ORDER BY clause to sort the result set based on one or more columns. For example:

SQL
SELECT column1, column2 FROM table_name ORDER BY column_name;

This will return results sorted in ascending order by default.

Default Sorting Direction

If no direction is specified after ORDER BY, MySQL assumes an ascending (ASC) sort. Explicitly specifying ASC or DESC for a particular column allows you to control the sorting behavior:

SQL
-- Ascending Order
SELECT * FROM employees ORDER BY salary;

-- Descending Order
SELECT * FROM employees ORDER BY salary DESC;
Multiple Columns Sorting

You can also specify multiple columns in ORDER BY and MySQL will sort based on those columns sequentially. For example, if you have a table with first name and last name:

SQL
SELECT * FROM people ORDER BY first_name, last_name;

This would first sort by first_name and then by last_name.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, the default sorting direction in MySQL for ORDER BY is ascending. However, it's good practice to always specify the direction (ASC or DESC) explicitly to avoid ambiguity.

#### Common mistakes
include forgetting to use ORDER BY when you need it, and not specifying a column name leading to unexpected results. Also, assuming that the default direction is descending can lead to confusion.

Common Follow-Ups
Can you write a query to sort employees by their last names in ascending order?
What happens if two rows have the same value for the sorted column? Does MySQL handle it?

**Code:**

```sql
-- Example of sorting with ORDER BY
SELECT * FROM employees ORDER BY salary DESC;
```

