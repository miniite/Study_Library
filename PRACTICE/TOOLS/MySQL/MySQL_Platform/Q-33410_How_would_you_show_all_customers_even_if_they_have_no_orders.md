---
type: "practice"
difficulty: "Intermediate"
---

# How would you show all customers even if they have no orders, and what join achieves that?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Basic Query Practice](<../../../../TOOLS/MySQL/MySQL_Platform/Basic_Query_Practice.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To demonstrate all customers in a database, even those without any orders, we need to perform an outer join operation on our customer and order tables. Specifically, using a LEFT JOIN ensures that every row from the left table (customers) appears in the result, with matching rows from the right table (orders). If there is no corresponding match, NULL values will fill in for the columns of the right table.

Let's break down this solution step by step:

Understand the Requirements: We want to display all customers, even those who haven't placed any orders.
Identify the Tables and Columns: Assume we have two tables: customers with columns like customer_id, name, etc., and orders with columns like order_id, customer_id, product, etc.
Perform an Outer Join: Use a LEFT JOIN to ensure that every customer is included in the result.

#### Triaging the Question

The question is probing your understanding of SQL joins, specifically focusing on outer joins (LEFT JOIN). The key assumptions are:

You understand the difference between inner and outer joins.
You know how to correctly write an SQL query using a LEFT JOIN.
You can handle the scenario where no matching records exist in one table.
Detailed Solution

We will perform a LEFT JOIN of the customers table with the orders table on the customer_id. Here's the step-by-step reasoning:

Identify the Tables and Columns:

customers: Has columns like customer_id, name, etc.
orders: Has columns like order_id, customer_id, product, etc.

#### Cons
truct the SQL Query:

SQL
SELECT
    c.customer_id,
    c.name,
    o.order_id,  -- Include order_id to show orders if any; otherwise, NULLs will appear.
    o.product
FROM
    customers AS c
LEFT JOIN
    orders AS o
ON
    c.customer_id = o.customer_id;
Explanation of the Query:
SELECT: Specifies which columns we want to include in the output.
FROM customers AS c: We are selecting from the customers table, aliasing it as c.
LEFT JOIN orders AS o ON c.customer_id = o.customer_id: This join ensures that every row from the customers table is included. If there is no matching row in the orders table, columns from o will be filled with NULLs.

#### Closing Summary
and Common Mistakes
Common Mistake: Using an INNER JOIN instead of a LEFT JOIN might result in missing customers without any orders.
Key Takeaway: Always use appropriate join types based on the requirement. For this case, LEFT JOIN is necessary to ensure all customers are shown.
Final Thoughts

Remember, understanding the differences between SQL joins and their implications can be crucial for data integrity and query optimization. A well-designed database query should always reflect the intended data retrieval logic accurately.

