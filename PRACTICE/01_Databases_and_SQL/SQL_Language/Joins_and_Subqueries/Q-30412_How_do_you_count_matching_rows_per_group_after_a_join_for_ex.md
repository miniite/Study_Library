---
type: "practice"
difficulty: "Intermediate"
---

# How do you count matching rows per group after a join, for example orders per customer?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Join Practice Problems](<../../../../01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries/Join_Practice_Problems.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the problem of counting matching rows per group after performing a join, let's start with a brief overview and then dive into the solution from first principles.

TL;DR:

The goal is to count the number of orders (or any other related rows) that match each customer. We will use SQL to perform this operation by leveraging JOIN and GROUP BY clauses.

#### Triaging the Question

The question asks us to count matching rows per group after a join, specifically in the context of counting orders per customer. The key elements are:

Join Operation: Combining data from two tables based on some condition.
Group By Clause: Organizing the result set into groups for aggregation functions like COUNT.
Aggregation Function: Using COUNT to calculate the number of matching rows.

#### Assumptions
We have two tables: customers and orders.
The customers table has columns such as customer_id, name.
The orders table has columns such as order_id, customer_id.

#### First-Principles Solution
Step 1: Define the Join

We need to join the customers and orders tables on a common column, typically customer_id. This will match each customer with their corresponding orders.

SQL
SELECT customers.customer_id, customers.name, COUNT(orders.order_id) AS order_count
FROM customers
LEFT JOIN orders ON customers.customer_id = orders.customer_id;
LEFT JOIN: Ensures that all customers are included in the result set even if they have no associated orders.
COUNT(orders.order_id): Counts the number of rows (orders) for each customer.
Step 2: Group by Customer

We group the results by customer_id and name, which gives us a count of orders per customer.

SQL
SELECT customers.customer_id, customers.name, COUNT(orders.order_id) AS order_count
FROM customers
LEFT JOIN orders ON customers.customer_id = orders.customer_id
GROUP BY customers.customer_id, customers.name;
Step 3: Handle Edge Cases
No Orders: Customers who have no orders will still be included in the result set with a count of 0.
Multiple Joins: If you need to join more tables, ensure that each table is appropriately joined and grouped.

#### Closing Summary
+ Common Mistakes/Follow-Ups

To effectively count matching rows per group after a join:

Use the Correct Join Type: Ensure the join type (e.g., LEFT JOIN, INNER JOIN) matches your requirement.
Group By All Key Columns: Always include all key columns in the GROUP BY clause to avoid errors and ensure correct aggregation.

#### Common Mistakes
Forgetting to use a JOIN to combine tables.
Not including necessary columns in the GROUP BY clause, leading to incorrect aggregations.
Using the wrong join type (e.g., using an INNER JOIN when you need a LEFT JOIN).

#### Example
SQL Query

Here's a worked example:

SQL
-- Sample data
CREATE TABLE customers (
    customer_id INT,
    name VARCHAR(255)
);

INSERT INTO customers VALUES (1, 'John Doe'), (2, 'Jane Smith');

CREATE TABLE orders (
    order_id INT,
    customer_id INT
);

INSERT INTO orders VALUES (101, 1), (102, 1), (103, 2);

-- Query to count orders per customer
SELECT customers.customer_id, customers.name, COUNT(orders.order_id) AS order_count
FROM customers
LEFT JOIN orders ON customers.customer_id = orders.customer_id
GROUP BY customers.customer_id, customers.name;

-- Result:
+------------+---------------+--------------+
| customer_id | name          | order_count  |
+------------+---------------+--------------+
| 1          | John Doe      | 2            |
| 2          | Jane Smith    | 1            |
+------------+---------------+--------------+
Self-Check

The solution is correct and complete.

**Comparison:**

| Table Name   | Columns        |
| ------------ | -------------- |
| customers    | customer_id, name |
| orders       | order_id, customer_id |

**Code:**

```sql
-- Sample SQL code for joins and subqueries
CREATE TABLE customers (
    customer_id INT,
    name VARCHAR(255)
);

INSERT INTO customers VALUES (1, 'John Doe'), (2, 'Jane Smith');

CREATE TABLE orders (
    order_id INT,
    customer_id INT
);

INSERT INTO orders VALUES (101, 1), (102, 1), (103, 2);

SELECT customers.customer_id, customers.name, COUNT(orders.order_id) AS order_count
FROM customers
LEFT JOIN orders ON customers.customer_id = orders.customer_id
GROUP BY customers.customer_id, customers.name;
```

**Example input / output:**

```text
+------------+---------------+--------------+
| customer_id | name          | order_count  |
+------------+---------------+--------------+
| 1          | John Doe      | 2            |
| 2          | Jane Smith    | 1            |
+------------+---------------+--------------+
```

**Data:**

```sql
-- Sample data for the tables
INSERT INTO customers VALUES (1, 'John Doe'), (2, 'Jane Smith');
INSERT INTO orders VALUES (101, 1), (102, 1), (103, 2);
```

**Flow:**

1. Customers
2. LEFT JOIN
3. Orders
4. GROUP BY customer_id, name
5. count(order_id) ASorder_count

