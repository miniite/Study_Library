---
type: "practice"
difficulty: "Basic"
---

# Write a query using GROUP BY to count the number of orders per customer.

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Basic Query Practice](<../../../../TOOLS/MySQL/MySQL_Platform/Basic_Query_Practice.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To write a query that counts the number of orders per customer in MySQL, we first need to understand the schema and basic requirements. The core idea is to group the data by the customer_id column and aggregate the count using the COUNT() function.

#### Triaging the Question

The question asks us to use SQL’s GROUP BY clause to perform an aggregation on a table that contains customer orders. We need to ensure we have a customers table, which typically includes fields like id, name, and orders. The goal is to return a list of unique customers along with the number of orders each has placed.

Step-by-Step Solution
Identify Table Schema: Assume there’s a customers table that has at least an id field for customer identification, and perhaps an order_id or similar field in another related orders table.
Join Tables if Necessary: If the orders are stored in a separate orders table linked by a foreign key to the customer_id, we need to join these tables.
Use GROUP BY Clause: Group the data by customer_id.
Aggregate Data with COUNT(): Use the COUNT() function on the relevant column (e.g., order_id) to get the count of orders per customer.
Full Query

Given that we have a table orders and it has a foreign key customer_id, the SQL query would look like this:

SQL
SELECT
    c.id AS customer_id,
    c.name AS customer_name,
    COUNT(o.order_id) AS order_count
FROM
    customers c
LEFT JOIN
    orders o ON c.id = o.customer_id
GROUP BY
    c.id, c.name;

#### Closing Summary
and Common Mistakes/Follow-Ups

This query will return a list of all unique customers with their associated number of orders. The LEFT JOIN ensures that even if a customer has no orders, they still appear in the result set, but with an order count of 0.

#### Common Mistakes

Not using GROUP BY on the correct column(s).
Forgetting to include the COUNT() function.
Missing the join condition between customers and orders tables (if necessary).

If asked further, discuss edge cases like handling no customer data or multiple foreign keys in different orders tables.

**Code:**

```sql
SELECT
    c.id AS customer_id,
    c.name AS customer_name,
    COUNT(o.order_id) AS order_count
FROM
    customers c
LEFT JOIN
    orders o ON c.id = o.customer_id
GROUP BY
    c.id, c.name;
```

