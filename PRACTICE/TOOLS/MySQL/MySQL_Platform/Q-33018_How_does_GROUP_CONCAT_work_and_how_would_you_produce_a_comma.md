---
type: "practice"
difficulty: "Intermediate"
---

# How does GROUP_CONCAT work and how would you produce a comma-separated list of a customer's product names?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ MySQL Query Basics](<../../../../TOOLS/MySQL/MySQL_Platform/MySQL_Query_Basics.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

GROUP_CONCAT is a powerful SQL function used to aggregate rows into a single string, which can be particularly useful in generating lists such as a customer's product names. In this answer, I'll explain how GROUP_CONCAT works and provide an example of producing a comma-separated list of a customer's product names.

Firstly, let’s break down what the question is asking:

What does GROUP_CONCAT do? It combines multiple rows into a single string with a specified delimiter.
How can we use it to get a list of products for a customer? We need a table structure that connects customers to their products and then use GROUP_CONCAT to aggregate the product names.

Now, let’s dive into the details:

#### Assumptions

There are two tables: customers (with columns customer_id, name) and orders (with columns order_id, product_name, customer_id).
Each order belongs to a customer.

Step-by-Step Solution:

Identify the relevant tables and fields.
Use INNER JOIN to match customers with their orders.
Apply GROUP_CONCAT to concatenate product names for each customer.

Here’s an example SQL query that achieves this:

SQL
SELECT
    c.customer_id,
    c.name,
    GROUP_CONCAT(o.product_name ORDER BY o.order_id SEPARATOR ', ') AS product_names
FROM
    customers c
INNER JOIN
    orders o ON c.customer_id = o.customer_id
GROUP BY
    c.customer_id, c.name;

This query groups the results by customer ID and name. For each group, it concatenates all associated product names with a comma and space as separators.

#### Closing Summary
GROUP_CONCAT is an excellent tool for generating lists in SQL, but it’s important to consider performance implications, especially when dealing with large datasets or numerous rows per group. Always ensure you have appropriate indexes on the join columns to optimize query execution.

#### Common Mistakes
and Follow-Ups:

Not ordering the concatenated products (remember to include ORDER BY in GROUP_CONCAT).
Forgetting to properly group by customer ID and name, which can lead to incorrect results or performance issues.

**Code:**

```sql
-- Example SQL query using GROUP_CONCAT
SELECT
    c.customer_id,
    c.name,
    GROUP_CONCAT(o.product_name ORDER BY o.order_id SEPARATOR ', ') AS product_names
FROM
    customers c
INNER JOIN
    orders o ON c.customer_id = o.customer_id
GROUP BY
    c.customer_id, c.name;
```

