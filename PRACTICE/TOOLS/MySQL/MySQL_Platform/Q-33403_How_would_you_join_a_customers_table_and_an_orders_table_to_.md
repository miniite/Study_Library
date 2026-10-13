---
type: "practice"
difficulty: "Intermediate"
---

# How would you join a customers table and an orders table to show each customer with their order count?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Basic Query Practice](<../../../../TOOLS/MySQL/MySQL_Platform/Basic_Query_Practice.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To join a customers table and an orders table in MySQL to display each customer along with their order count, we need to use SQL's JOIN operations combined with aggregate functions. The key idea here is to group the orders by the customer ID and then count them.

The question is probing your understanding of:

How to join tables using INNER JOIN or other types like LEFT JOIN.
Use of GROUP BY clause for aggregation.
Application of COUNT() function in SQL queries.

We'll assume that the customers table has a primary key column named customer_id, and the orders table also has an order-related ID (let's call it order_id) along with a foreign key to customer_id. Here’s how you can achieve this:

Join Tables: First, join the customers and orders tables on the common customer_id.
Group by Customer: Use the GROUP BY clause to group the result set by each customer.
Count Orders per Customer: Apply the COUNT() function within a SELECT statement to count orders for each customer.

Let's break it down step-by-step:

SQL
-- Step 1: Perform an INNER JOIN on the customer_id field in both tables
SELECT c.customer_id, c.customer_name, COUNT(o.order_id) AS order_count
FROM customers c
INNER JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.customer_name;

This query will give you a list of all customers and the count of their respective orders. If there are no orders for some customers, those rows won't appear in the result (due to the INNER JOIN).

**Example input / output:**

```text
Input:
+---------+-------------+
| customer_id | customer_name |
+---------+-------------+
| 1        | Alice        |
| 2        | Bob          |
| 3        | Carol        |

+---------+----------+
| order_id |
+---------+----------+
| 101      |
| 101      |
| 103      |
| 104      |
| 105      |

Expected Output:
+---------+-------------+------------+
| customer_id | customer_name | order_count|
+---------+-------------+------------+
| 1         | Alice        | 2          |
| 2         | Bob          | 1          |
| 3         | Carol        | 0          |
```

