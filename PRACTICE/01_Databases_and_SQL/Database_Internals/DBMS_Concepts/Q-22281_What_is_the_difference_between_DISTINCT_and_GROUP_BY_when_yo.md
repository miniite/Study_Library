---
type: "practice"
difficulty: "Basic"
---

# What is the difference between DISTINCT and GROUP BY when you just want unique rows?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1 · [↑ WHERE vs HAVING](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/WHERE_vs_HAVING.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To answer the question of what differentiates DISTINCT from GROUP BY in SQL, let's first summarize the core concepts and then delve into a detailed explanation.

TL;DR:
DISTINCT: This clause filters duplicate rows in a single column or multiple columns.
GROUP BY: This clause groups rows that have the same values in specified columns into summary rows.
In scenarios where you want unique rows, DISTINCT is straightforward and operates on individual rows. However, GROUP BY provides more flexibility by allowing aggregate functions and complex operations.

#### Triaging the Question

The question asks about the difference between using DISTINCT versus GROUP BY when aiming to retrieve unique rows from a database table. The key points are understanding how each clause works and their implications on query results.

#### First-Principles Solution
DISTINCT
Purpose: To eliminate duplicate values in a result set.
Syntax: SELECT column_name(s) FROM table_name WHERE condition GROUP BY column_name(s);
Behavior: It filters out any duplicates, making the rows unique. If used without an aggregate function, it operates on individual rows.

#### Example
SQL
SELECT DISTINCT column1, column2 FROM table_name;
GROUP BY
Purpose: To group results based on a specified criterion and perform aggregations.
Syntax: SELECT column(s), aggregation_function(column) FROM table_name WHERE condition GROUP BY column(s);
Behavior: It groups rows that have the same values in specified columns into summary rows. You can use aggregate functions like COUNT, SUM, AVG, etc., to compute summaries for each group.

#### Example
SQL
SELECT column1, COUNT(column2) FROM table_name GROUP BY column1;
Detailed Explanation:
DISTINCT

DISTINCT is a simple and direct way to filter out duplicate rows. It acts on the entire row or selected columns after the FROM clause. For example:

If you have a table with customer orders, using SELECT DISTINCT order_id, customer_name FROM customers; will return only unique combinations of order_id and customer_name.
GROUP BY

GROUP BY, while primarily used for grouping rows based on one or more columns, can also be leveraged to get unique values by combining it with aggregate functions. For instance:

Using the same table as above, SELECT customer_name FROM customers GROUP BY customer_name; will give you a list of unique customer_names. However, this is essentially a shortcut for using both GROUP BY and DISTINCT.
A more powerful use case would be combining GROUP BY with aggregate functions to get meaningful summaries:
SQL
SELECT customer_name, COUNT(*) FROM customers GROUP BY customer_name;
This query not only filters out duplicate rows but also counts the number of orders per customer.

#### Closing Summary
and Common Mistakes/Follow-Ups
Common Mistake: Using GROUP BY when you want unique rows. Remember that GROUP BY is typically used for aggregation, so using it just to get unique values can lead to confusion.

#### Follow-Up
Question: When should one prefer DISTINCT over GROUP BY, and vice versa? Generally, use DISTINCT if your goal is purely filtering duplicates without needing aggregate functions. Use GROUP BY when you want to perform aggregations on grouped data or when the unique values are part of a larger analytical query.

#### Example
Queries
SQL
-- Using DISTINCT
SELECT DISTINCT order_id, customer_name FROM customers;

-- Using GROUP BY with count
SELECT customer_name, COUNT(*) FROM customers GROUP BY customer_name;

