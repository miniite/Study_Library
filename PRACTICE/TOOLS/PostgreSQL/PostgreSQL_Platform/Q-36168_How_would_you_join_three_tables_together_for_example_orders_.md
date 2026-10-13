---
type: "practice"
difficulty: "Intermediate"
---

# How would you join three tables together, for example orders, customers, and products?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Many-to-many Modeling](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Many-to-many_Modeling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To join three tables in PostgreSQL—specifically, orders, customers, and products—we need to understand the relationships between these tables and use appropriate SQL commands to retrieve the desired data. This task is fundamental for relational database operations and will help ensure that we fetch the correct data while optimizing performance.

The question asks us to join three tables based on their common fields, which typically means using foreign keys. For this example, let's assume:

orders has a foreign key customer_id linking it to the customers table.
products has a foreign key order_id linking it back to the orders table.

Now, I'll walk you through joining these tables step-by-step from first principles.

Firstly, let's consider the basic SQL syntax for JOIN operations:

SQL
SELECT *
FROM orders o
JOIN customers c ON o.customer_id = c.id
JOIN products p ON o.order_id = p.order_id;

However, this might not be accurate as order_id in products should likely refer to customer_id or directly match the order details. Let's assume a more typical setup where:

orders has columns: id, customer_id, and product_id.
customers has columns: id, name.
products has columns: id, name.

Given this, the correct SQL query would be:

SQL
SELECT o.id AS order_id,
       c.name AS customer_name,
       p.name AS product_name,
       o.quantity
FROM orders o
JOIN customers c ON o.customer_id = c.id
JOIN products p ON o.product_id = p.id;

This query will return a result set with columns for order_id, customer_name, product_name, and the quantity of each item in the order.

**Code:**

```sql
SELECT o.id AS order_id,
       c.name AS customer_name,
       p.name AS product_name,
       o.quantity
FROM orders o
JOIN customers c ON o.customer_id = c.id
JOIN products p ON o.product_id = p.id;
```

**Example input / output:**

```text
+---------+------------+--------------+--------+
| order_id | customer_name | product_name | quantity |
+---------+------------+--------------+--------+
| 1       | Alice Smith  | Laptop       | 1       |
| 2       | Bob Johnson  | Headphones   | 3       |
| 3       | Carol Lee    | Mouse        | 2       |
+---------+------------+--------------+--------+
```

