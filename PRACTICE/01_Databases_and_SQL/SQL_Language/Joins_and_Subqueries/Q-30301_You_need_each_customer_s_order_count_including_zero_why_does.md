---
type: "practice"
difficulty: "Intermediate"
---

# You need each customer's order count including zero — why does a LEFT JOIN + COUNT(*) overcount and how does COUNT(child.id) fix it?

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ Join Practice Problems](<../../../../01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries/Join_Practice_Problems.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with databases, especially in scenarios involving multiple tables (like customers and orders), understanding joins and subqueries is crucial. This problem involves using a LEFT JOIN to ensure that each customer has an entry in the result set, even if they have no orders. The challenge lies in accurately counting orders for each customer without overcounting.

TL;DR

A LEFT JOIN + COUNT(*) can lead to overcounting because it includes null values from the right table (orders) in the count. Using COUNT(child.id) instead ensures that only non-null order IDs are counted, giving an accurate result.

#### Triaging the Question

The question is asking you to explain why using a LEFT JOIN with COUNT(*) overcounts and how to use COUNT(child.id) to get the correct count of orders for each customer. The key here is understanding that LEFT JOIN pairs every row from the left table (customers) with matching rows in the right table (orders). When no match exists, it still adds a row but with null values on the side without data.

#### Assumptions
and First-Principles Solution
Schema: We have two tables: customers and orders.
Objective: To get each customer's order count including those with zero orders.
Overcounting Issue: LEFT JOIN + COUNT(*) overcounts because it counts null values as well.

Let's break this down step-by-step:

Schema Overview:

SQL
customers (id, name)
orders (id, customer_id)

Initial Query with LEFT JOIN and COUNT(*):

SQL
SELECT
    c.id AS customer_id,
    c.name,
    COUNT(o.id) AS order_count
FROM
    customers c
LEFT JOIN
    orders o ON c.id = o.customer_id
GROUP BY
    c.id;

This query will include every customer, even those with no orders. However, the COUNT(*) includes all rows (including nulls from unmatched joins), leading to overcounting.

Correct Query with COUNT(child.id):

SQL
SELECT
    c.id AS customer_id,
    c.name,
    COUNT(o.id) AS order_count
FROM
    customers c
LEFT JOIN
    orders o ON c.id = o.customer_id
GROUP BY
    c.id;

This query avoids overcounting by explicitly counting only the non-null id from the orders table.

#### Closing Summary
and Common Mistakes/Follow-Ups

To summarize, using LEFT JOIN + COUNT(*) can lead to overcounting because it includes null values in the count. The correct approach is to use COUNT(child.id) to ensure that only actual order entries are counted. This ensures accurate counts for each customer, even when they have no orders.

#### Common Mistakes

Forgetting to explicitly filter out nulls when counting.
Misunderstanding the behavior of LEFT JOIN in combination with aggregate functions.

#### Follow-Ups

Discuss additional techniques or optimizations if needed (e.g., using COALESCE).
Explore alternative approaches, such as subqueries or window functions, for more complex scenarios.

**Comparison:**

| customer_id | name  | order_count (LEFT JOIN) | order_count (Correct Query) |
|-------------|-------|------------------------|----------------------------|
| 1           | Alice | 2                      | 1                          |
| 2           | Bob   | 0                      | 0                          |

**Code:**

```sql
-- Example query with LEFT JOIN and COUNT(*)
SELECT
    c.id AS customer_id,
    c.name,
    COUNT(o.id) AS order_count
FROM
    customers c
LEFT JOIN
    orders o ON c.id = o.customer_id
GROUP BY
    c.id;

-- Corrected query using COUNT(child.id)
SELECT
    c.id AS customer_id,
    c.name,
    COUNT(o.id) AS order_count
FROM
    customers c
LEFT JOIN
    orders o ON c.id = o.customer_id
GROUP BY
    c.id;
```

**Example input / output:**

```

Input:

SQL
Copy
-- Customers Table
INSERT INTO customers (id, name) VALUES
    (1, 'Alice'),
    (2, 'Bob');

-- Orders Table
INSERT INTO orders (id, customer_id) VALUES
    (1, 1),
    (2, 1);

Output:

SQL
Copy
SELECT
    c.id AS customer_id,
    c.name,
    COUNT(o.id) AS order_count
FROM
    customers c
LEFT JOIN
    orders o ON c.id = o.customer_id
GROUP BY
    c.id;

-- Result:
+------------+-------+--------------+
| customer_id | name  | order_count  |
+------------+-------+--------------+
|          1  | Alice |            2  |
|          2  | Bob   |            0  |
+------------+-------+--------------+

Corrected Query:

SQL
Copy
SELECT
    c.id AS customer_id,
    c.name,
    COUNT(o.id) AS order_count
FROM
    customers c
LEFT JOIN
    orders o ON c.id = o.customer_id
GROUP BY
    c.id;

-- Result:
+------------+-------+--------------+
| customer_id | name  | order_count  |
+------------+-------+--------------+
|          1  | Alice |            1  |
|          2  | Bob   |            0  |
+------------+-------+--------------+
```

**Data:**

```sql
CREATE TABLE customers (
    id INT PRIMARY KEY,
    name VARCHAR(255)
);

CREATE TABLE orders (
    id INT PRIMARY KEY,
    customer_id INT,
    FOREIGN KEY (customer_id) REFERENCES customers(id)
);
```

**Flow:**

1. Customers Table
2. LEFT JOIN Orders
3. Count(*) Overcounting
4. Result Includes Nulls
5. Correct Query
6. Count(o.id)
7. Non-null Order Counts Only
8. Accurate Results

