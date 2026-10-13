---
type: "practice"
difficulty: "Intermediate"
---

# You need each customer's first purchase date shown on every one of their order rows; which window function and frame would you use?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ First Last per Group and Running Totals](<../../../../01_Databases_and_SQL/SQL_Language/Window_Functions/First_Last_per_Group_and_Running_Totals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this problem, we need to append the first purchase date of each customer to every row in their orders table. Window functions are perfect for such tasks because they allow us to perform calculations across a set of rows that are related to the current row.

#### Triage

The core question is asking how to use window functions to add metadata (the first purchase date) from one column to all rows associated with each customer in another column. Specifically, we need to identify and propagate the earliest purchase date for each customer across all their orders.

#### First-Principles Solution
Identify the Customer and Order Dates: We start by identifying unique customers along with their corresponding order dates.
Determine the Earliest Purchase Date per Customer: Use a window function to find the minimum (or earliest) purchase date for each customer.
Propagate the First Purchase Date to All Orders: Append this first purchase date to every row in the orders table.
Step-by-Step Solution

Identify Unique Customers and Order Dates:

SQL
SELECT customer_id, order_date
FROM orders;

Determine the Earliest Purchase Date per Customer using Window Functions:

We can use ROW_NUMBER() to rank rows within a window partitioned by customer_id based on order_date.
The first row in each partition will have the earliest purchase date.
SQL
WITH customer_first_purchase AS (
    SELECT
        customer_id,
        order_date,
        ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY order_date ASC) as rn
    FROM orders
)
SELECT
    customer_id,
    order_date
FROM customer_first_purchase
WHERE rn = 1;

Join the First Purchase Date Back to Orders:

Join the first purchase date back to the original orders table on customer_id.
Use a left join or window function to ensure every row gets the earliest purchase date.
SQL
WITH customer_first_purchase AS (
    SELECT
        customer_id,
        order_date,
        ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY order_date ASC) as rn
    FROM orders
)
SELECT
    o.customer_id,
    o.order_id,
    o.order_date,
    fp.order_date AS first_purchase_date
FROM orders o
LEFT JOIN (
    SELECT
        customer_id,
        order_date
    FROM customer_first_purchase
    WHERE rn = 1
) fp ON o.customer_id = fp.customer_id;

#### Closing Summary

By using the ROW_NUMBER() window function and a subsequent join, we can effectively append each customer's first purchase date to every one of their order rows. This approach ensures that all relevant information is preserved and easily accessible for further analysis.

#### Common Mistakes
and Follow-Ups
Mistake: Forgetting to use the ROW_NUMBER() function or incorrectly partitioning the window.

#### Follow-Up
Discuss how different window functions like RANK(), DENSE_RANK(), etc., might be used in similar scenarios, but with subtle differences in behavior.

**Comparison:**

```
orders
First Purchase Date
customer_first_purchase
Customer ID
order_date
```

**Code:**

```sql
WITH customer_first_purchase AS (
    SELECT
        customer_id,
        order_date,
        ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY order_date ASC) as rn
    FROM orders
)
SELECT
    o.customer_id,
    o.order_id,
    o.order_date,
    fp.order_date AS first_purchase_date
FROM orders o
LEFT JOIN (
    SELECT
        customer_id,
        order_date
    FROM customer_first_purchase
    WHERE rn = 1
) fp ON o.customer_id = fp.customer_id;
```

**Example input / output:**

```sql
WITH customer_first_purchase AS (
    SELECT
        customer_id,
        order_date,
        ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY order_date ASC) as rn
    FROM orders
)
SELECT
    o.customer_id,
    o.order_id,
    o.order_date,
    fp.order_date AS first_purchase_date
FROM orders o
LEFT JOIN (
    SELECT
        customer_id,
        order_date
    FROM customer_first_purchase
    WHERE rn = 1
) fp ON o.customer_id = fp.customer_id;
customer_id	order_id	order_date	first_purchase_date
101	1001	2023-01-05	2023-01-05
101	1002	2023-01-10	2023-01-05
102	1003	2023-02-07	2023-02-07
102	1004	2023-02-14	2023-02-07
103	1005	2023-03-01	2023-03-01
```

**Data:**

```sql
CREATE TABLE orders (
    customer_id INT,
    order_id INT,
    order_date DATE
);
INSERT INTO orders (customer_id, order_id, order_date)
VALUES
(101, 1001, '2023-01-05'),
(101, 1002, '2023-01-10'),
(102, 1003, '2023-02-07'),
(102, 1004, '2023-02-14'),
(103, 1005, '2023-03-01');
```

**Flow:**

1. Read Orders Table
2. Apply ROW_NUMBER()Window Function
3. Determine First Purchase perCustomer
4. Join Back to Original Table
5. Propagate First Purchase Date

