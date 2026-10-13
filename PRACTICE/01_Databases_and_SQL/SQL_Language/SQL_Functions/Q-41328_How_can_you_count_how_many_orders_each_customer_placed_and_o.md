---
type: "practice"
difficulty: "Intermediate"
---

# How can you count how many orders each customer placed and only show those with more than 5 orders?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ UDF Performance](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Functions/UDF_Performance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve this problem, we need to write a SQL query that aggregates the number of orders per customer and filters out customers who have made fewer than or equal to 5 orders. Let's break down the solution step by step.

First, understand what the question is asking:

We need to count the number of orders each customer has placed.
We should only display customers who have placed more than 5 orders.

Now let’s outline the steps from first principles:

Aggregate Orders: Use a GROUP BY clause to group records by customer ID and count the number of orders for each customer.
Filter Customers: Add a HAVING clause to filter out customers who do not meet our condition (more than 5 orders).

Here’s how we can implement this in SQL:

SQL
SELECT customer_id, COUNT(*) AS order_count
FROM orders
GROUP BY customer_id
HAVING COUNT(*) > 5;

This query groups the orders table by customer_id, counts the number of rows for each group (which corresponds to the number of orders), and then filters out any customers who have placed 5 or fewer orders using the HAVING clause.

**Code:**

```sql
SELECT customer_id, COUNT(*) AS order_count
FROM orders
GROUP BY customer_id
HAVING COUNT(*) > 5;
```

**Example input / output:**

```text
This section would normally contain sample data and the resulting output. Since this is a theoretical question, no specific input-output examples are provided.
```

**Flow:**

1. SELECT customer_id, COUNT(*)AS order_count
2. FROM orders
3. GROUP BY customer_id
4. HAVING COUNT(*) > 5

