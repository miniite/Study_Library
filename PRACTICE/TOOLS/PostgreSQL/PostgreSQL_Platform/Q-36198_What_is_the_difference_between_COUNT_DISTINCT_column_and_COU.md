---
type: "practice"
difficulty: "Intermediate"
---

# What is the difference between COUNT(DISTINCT column) and COUNT(column) and when would the results differ?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ PostgreSQL Query Basics](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/PostgreSQL_Query_Basics.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand the differences between COUNT(DISTINCT column) and COUNT(column) in PostgreSQL, it's essential to grasp how these SQL functions behave with respect to duplicate values within a column. The key difference lies in their handling of duplicates—COUNT(column) counts all occurrences of the column value, while COUNT(DISTINCT column) only counts unique values.

#### Triaging the Question

The question is asking for a detailed explanation of two commonly used SQL functions and scenarios where they produce different results. The interviewer is likely probing your understanding of basic SQL operations, data redundancy handling, and the implications of these differences in query outcomes. Assumptions include familiarity with PostgreSQL syntax and its typical use cases.

#### First-Principles Solution

Let's start from the basics:

COUNT(column):

This function counts every row where column has a non-null value.

#### Example
If you have an orders table, SELECT COUNT(order_id) FROM orders; will return the total number of rows in the table (excluding rows with NULL values in order_id).

COUNT(DISTINCT column):

This function returns the count of unique non-null values in the specified column.

#### Example
Using the same orders table, SELECT COUNT(DISTINCT customer_id) FROM orders; will return a number representing the distinct customers who have placed an order.
Detailed Explanation

To illustrate this with a concrete example:

Sample Data:
PLAINTEXT
orders (table)
+------------+
| order_id   |
+------------+
| 101        |
| 102        |
| 103        |
| 104        |
| 105        |
+------------+
customers (table)
+------------+
| customer_id|
+------------+
| 1          |
| 1          |
| 2          |
| 3          |
| 3          |
+------------+
Query Execution:

SELECT COUNT(order_id) FROM orders; will return 5.

SELECT COUNT(DISTINCT order_id) FROM orders; will also return 5 because all order IDs are unique.

Now, consider a different scenario:

PLAINTEXT
orders (table)
+------------+
| order_id   | customer_id |
+------------+
| 101        | 1           |
| 102        | 1           |
| 103        | 2           |
| 104        | 3           |
| 105        | 3           |
+------------+
SELECT COUNT(order_id) FROM orders; will return 5.
SELECT COUNT(DISTINCT customer_id) FROM orders; will return 3, as there are three distinct customers (1, 2, and 3).

#### Common Mistakes
and Follow-Ups

#### Common mistakes
include:

Confusing the behavior of these functions with other aggregate functions like SUM or AVG.
Not considering NULL values in your COUNT function, which can lead to unexpected results.

#### Follow-ups
might include discussing performance considerations between the two queries, especially for large datasets, or exploring alternative ways to achieve similar results using window functions or subqueries.

#### Closing Summary

Understanding the nuances between COUNT(column) and COUNT(DISTINCT column) is crucial for writing efficient SQL queries. COUNT(column) should be used when you need to count all occurrences of a value in a column, while COUNT(DISTINCT column) should be used when you specifically need the number of unique values.

**Comparison:**

```
tableDiagram
    customers -->|JOIN|-- orders
    orders
    customers
```

**Code:**

```sql
-- Example query
SELECT COUNT(order_id) AS total_orders,
       COUNT(DISTINCT customer_id) AS distinct_customers
FROM orders;
```

