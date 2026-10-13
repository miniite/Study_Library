---
type: "practice"
difficulty: "Basic"
---

# How would you check for duplicate rows in a table using SQL?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Uniqueness Checks](<../../../../02_Data_Engineering/Data_Quality/Validation/Uniqueness_Checks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To check for duplicate rows in a SQL table, we need to identify any rows that have identical values across all columns or specific columns of interest. The basic idea is to use aggregate functions and window functions to identify duplicates based on unique key(s) or combinations of column values.

#### Triaging the Question

The question is asking how to detect duplicate rows in a database table using SQL. This involves understanding the concept of data quality and validation, specifically focusing on identifying and handling redundant entries. The interviewer might be probing your knowledge of SQL syntax, aggregate functions like COUNT(), GROUP BY clauses, and window functions.

#### First-Principles Solution
Identify Unique Keys: Determine which columns or combination of columns could serve as unique keys.
Use Aggregate Functions: Utilize the COUNT() function to count occurrences of each row based on these key(s).
Filter for Duplicates: Use a WHERE clause with a HAVING condition to filter out rows that appear more than once.
Step-by-Step Solution

Let's assume we have a table named orders with columns: order_id, customer_id, product_name, and quantity.

1. Identify Unique Keys

For simplicity, let's consider order_id as the unique key for this example.

2. Use Aggregate Functions

We can use the COUNT() function in combination with GROUP BY to count occurrences of each order ID.

SQL
SELECT order_id, COUNT(*) AS duplicate_count
FROM orders
GROUP BY order_id;

This query will return a list where each row represents an order_id and its corresponding number of occurrences.

3. Filter for Duplicates

To get only the rows that are duplicates (i.e., those with more than one occurrence), we add a HAVING clause:

SQL
SELECT order_id, COUNT(*) AS duplicate_count
FROM orders
GROUP BY order_id
HAVING COUNT(*) > 1;

This will give us all order_ids that appear more than once in the table.

Full Solution and Closing Summary

The full SQL solution involves identifying unique keys, using aggregate functions to count occurrences, and filtering results based on duplicate counts. In our example with the orders table:

SQL
SELECT order_id, COUNT(*) AS duplicate_count
FROM orders
GROUP BY order_id
HAVING COUNT(*) > 1;

This will return a list of all order_ids that have more than one entry in the table.

#### Common Mistakes
and Follow-Ups
Mistake: Using only the COUNT() function without grouping. This won't give you the actual duplicate rows.

#### Follow-Up
Discuss how to handle cases where multiple columns might define uniqueness (e.g., a combination of customer_id and product_name). You could use a similar approach but with multiple columns in the GROUP BY clause.

**Code:**

```sql
-- SQL query for finding duplicate rows based on order_id
SELECT order_id, COUNT(*) AS duplicate_count
FROM orders
GROUP BY order_id
HAVING COUNT(*) > 1;
```

