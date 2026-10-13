---
type: "practice"
difficulty: "Intermediate"
---

# How do you join a users table with an orders table to build a per-user feature set for a churn model?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Validating Join Cardinality](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Validating_Join_Cardinality.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve this problem, we need to combine user information from the users table with their purchase history from the orders table to create a per-user feature set suitable for a churn prediction model. This involves understanding the schema of both tables and applying appropriate SQL joins based on common fields such as user IDs.

#### Triaging the Question

The question is asking how to join two relational database tables (users and orders) where users can have multiple orders, into a single dataset that includes relevant features for each user. The key is to ensure we capture all necessary information about the user while accurately reflecting their purchasing behavior over time.

Key Assumptions
The users table contains user IDs and other relevant demographic or behavioral data.
The orders table contains order IDs, user IDs, order dates, and possibly other details like order amounts.

#### First Principles Solution
Understand the Schema:
Let's assume:
SQL
users (user_id, name, email, registration_date)
orders (order_id, user_id, purchase_date, total_amount)
Determine the Join Condition:
Since both tables have a user_id field, we can join them using this common key.
Decide on the Type of Join:
A left join is typically used here to ensure all users are included even if they have no orders.
Choose Relevant Features:
Common features might include number of orders, total spend, average order value, and time since last purchase.
SQL Query
SQL
SELECT
    u.user_id,
    u.name,
    COUNT(o.order_id) AS num_orders,
    SUM(o.total_amount) AS total_spend,
    AVG(o.total_amount) AS avg_order_value,
    MAX(o.purchase_date) - u.registration_date AS days_since_last_purchase
FROM
    users u
LEFT JOIN
    orders o ON u.user_id = o.user_id
GROUP BY
    u.user_id, u.name;

#### Closing Summary
and Common Mistakes/Follow-Ups

This query effectively combines the user data with their order history. However, there are a few common pitfalls:

Ensure the purchase_date is properly converted to a date format.

#### Cons
ider adding more features like frequency of orders or average time between purchases for a richer feature set.

#### Common Mistakes
Failing to Use LEFT JOIN: This might exclude users without any orders, leading to incomplete data.
Incorrect Date Calculations: Ensure purchase_date and registration_date are in the correct date format.
Ignoring Null Values: Properly handle null values when calculating features like days_since_last_purchase.

#### Example
Data
SQL
-- Users Table
CREATE TABLE users (
    user_id INT PRIMARY KEY,
    name VARCHAR(255),
    email VARCHAR(255),
    registration_date DATE
);

INSERT INTO users (user_id, name, email, registration_date) VALUES
(1, 'John Doe', 'john@example.com', '2020-01-01'),
(2, 'Jane Smith', 'jane@example.com', '2021-03-15');

-- Orders Table
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    user_id INT,
    purchase_date DATE,
    total_amount DECIMAL(10, 2)
);

INSERT INTO orders (order_id, user_id, purchase_date, total_amount) VALUES
(1, 1, '2020-06-15', 59.99),
(2, 1, '2020-07-31', 34.99),
(3, 2, '2021-04-01', 89.99);
Worked Input->Output
SQL
-- Query Output
+---------+-----------+------------+-------------+---------------------+
| user_id | name      | num_orders | total_spend | days_since_last_purchase |
+---------+-----------+------------+-------------+---------------------+
| 1       | John Doe  | 2          | 94.98       | 358                  |
| 2       | Jane Smith| 1          | 89.99       | 367                  |
+---------+-----------+------------+-------------+---------------------+

**Comparison:**

| user_id | name      | num_orders | total_spend | avg_order_value | days_since_last_purchase |
| ------- | --------- | ---------- | ----------- | -------------- | ----------------------- |
| 1       | John Doe  | 2          | 94.98       | 47.49          | 358                     |
| 2       | Jane Smith| 1          | 89.99       | 89.99          | 367                     |

**Code:**

```sql
SELECT
    u.user_id,
    u.name,
    COUNT(o.order_id) AS num_orders,
    SUM(o.total_amount) AS total_spend,
    AVG(o.total_amount) AS avg_order_value,
    MAX(o.purchase_date) - u.registration_date AS days_since_last_purchase
FROM
    users u
LEFT JOIN
    orders o ON u.user_id = o.user_id
GROUP BY
    u.user_id, u.name;
```

**Example input / output:**

```text
-- Query Output
+---------+-----------+------------+-------------+---------------------+
| user_id | name      | num_orders | total_spend | days_since_last_purchase |
+---------+-----------+------------+-------------+---------------------+
| 1       | John Doe  | 2          | 94.98       | 358                  |
| 2       | Jane Smith| 1          | 89.99       | 367                  |
+---------+-----------+------------+-------------+---------------------+
```

**Data:**

```sql
-- Users Table Data
CREATE TABLE users (
    user_id INT PRIMARY KEY,
    name VARCHAR(255),
    email VARCHAR(255),
    registration_date DATE
);

INSERT INTO users (user_id, name, email, registration_date) VALUES
(1, 'John Doe', 'john@example.com', '2020-01-01'),
(2, 'Jane Smith', 'jane@example.com', '2021-03-15');

-- Orders Table Data
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    user_id INT,
    purchase_date DATE,
    total_amount DECIMAL(10, 2)
);

INSERT INTO orders (order_id, user_id, purchase_date, total_amount) VALUES
(1, 1, '2020-06-15', 59.99),
(2, 1, '2020-07-31', 34.99),
(3, 2, '2021-04-01', 89.99);
```

**Flow:**

1. Output
2. common key
3. (user_id)
4. user_id
5. (per-user feature set)

