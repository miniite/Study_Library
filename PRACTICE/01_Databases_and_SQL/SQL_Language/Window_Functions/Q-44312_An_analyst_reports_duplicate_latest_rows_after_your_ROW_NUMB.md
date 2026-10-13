---
type: "practice"
difficulty: "Intermediate"
---

# An analyst reports duplicate 'latest' rows after your ROW_NUMBER dedup; walk through how you'd reproduce it and what non-unique ORDER BY key is the usual root cause.

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Deterministic Deduplication](<../../../../01_Databases_and_SQL/SQL_Language/Window_Functions/Deterministic_Deduplication.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this issue, let's first understand that we are dealing with a common problem in data processing where rows marked as 'latest' by a ROW_NUMBER() function still appear duplicated after deduplication. The root cause typically lies in the ORDER BY key used to determine the latest row.

#### Triage

The question is probing your understanding of how window functions and ordering keys work together, particularly when dealing with time-based or versioned data. We need to verify that our ORDER BY clause is capturing all relevant criteria that define a 'latest' record. Common mistakes include missing critical fields in the order by, leading to duplicate latest records.

#### First-Principles Solution

Let's break down the solution step-by-step:

Identify the Issue: The analyst reports duplicated rows marked as the latest.
Understand ROW_NUMBER() Function: This function assigns a unique number to each row within a partition of a result set, based on the order specified by the ORDER BY clause. If the same value appears for multiple rows in the ORDER BY column(s), they will receive the same row number.
Check the ORDER BY Key: The issue likely stems from an incomplete or incorrect ORDER BY key that does not account for all necessary fields.
Reproduce the Issue

To reproduce this, we can follow these steps:

Sample Data Schema:

SQL
CREATE TABLE orders (
    order_id INT,
    customer_id INT,
    order_date DATE,
    amount DECIMAL(10, 2),
    version INT -- version number for handling updates
);

Initial Query with Incorrect ORDER BY Key (Duplicate Latest):

SQL
WITH latest_orders AS (
    SELECT
        order_id,
        customer_id,
        order_date,
        amount,
        ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY order_date DESC) as rn
    FROM orders
)
SELECT * FROM latest_orders WHERE rn = 1;

Dry Run: Assume the following sample data:

SQL
INSERT INTO orders (order_id, customer_id, order_date, amount, version) VALUES
(1, 1001, '2023-01-01', 50.00, 1),
(2, 1001, '2023-02-01', 55.00, 2),
(3, 1001, '2023-03-01', 60.00, 3),
(4, 1002, '2023-01-01', 70.00, 1),
(5, 1002, '2023-02-01', 75.00, 2);

Expected Output:

SQL
order_id | customer_id | order_date     | amount | rn
-------------------------------------------------
   3    |      1001   | '2023-03-01'   | 60.00  | 1
   5    |      1002   | '2023-02-01'   | 75.00  | 1

However, if we missed a versioning key: sql     WITH latest_orders AS (         SELECT              order_id,             customer_id,             order_date,             amount,             ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY order_date DESC) as rn         FROM orders     )     SELECT * FROM latest_orders WHERE rn = 1;

#### Common Mistakes
and Follow-Ups
Missing Versioning Key: If the order_date is not unique for a given customer_id, and no versioning key (like version in our schema) is used, multiple rows can be marked as 'latest'.
Incorrect ORDER BY Clause: Ensure that all fields relevant to determining the latest row are included.
Multiple ROW_NUMBER() Functions: Using multiple window functions or complex partitioning without proper ORDER BY.

#### Closing Summary

In summary, when dealing with duplicate 'latest' rows using ROW_NUMBER(), it is crucial to ensure your ORDER BY clause captures all necessary fields that define a unique latest record. Common mistakes include missing versioning keys and incomplete ORDER BY clauses.

Self-Check

This solution covers the core issue, potential mistakes, and provides a clear example of how to reproduce and troubleshoot the problem.

**Comparison:**

```
order_id	customer_id	order_date	amount	rn
3	1001	2023-03-01	60.00	1
5	1002	2023-02-01	75.00	1
order_id	customer_id	order_date	amount	rn
3	1001	2023-03-01	60.00	1
5	1002	2023-02-01	75.00	1
```

**Code:**

```sql
WITH latest_orders AS (
    SELECT
        order_id,
        customer_id,
        order_date,
        amount,
        ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY order_date DESC, version DESC) as rn
    FROM orders
)
SELECT * FROM latest_orders WHERE rn = 1;
```

**Example input / output:**

```text
WITH latest_orders AS (
    SELECT
        order_id,
        customer_id,
        order_date,
        amount,
        ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY order_date DESC) as rn
    FROM orders
)
SELECT * FROM latest_orders WHERE rn = 1;

-- Expected Output:
-- order_id | customer_id | order_date     | amount | rn
-- -------------------------------------------------
--       3   |      1001   | '2023-03-01'   | 60.00  | 1
--       5   |      1002   | '2023-02-01'   | 75.00  | 1

WITH latest_orders AS (
    SELECT
        order_id,
        customer_id,
        order_date,
        amount,
        ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY order_date DESC, version DESC) as rn
    FROM orders
)
SELECT * FROM latest_orders WHERE rn = 1;

-- Corrected Output:
-- order_id | customer_id | order_date     | amount | rn
-- -------------------------------------------------
--       3   |      1001   | '2023-03-01'   | 60.00  | 1
--       5   |      1002   | '2023-02-01'   | 75.00  | 1
```

**Data:**

```sql
CREATE TABLE orders (
    order_id INT,
    customer_id INT,
    order_date DATE,
    amount DECIMAL(10, 2),
    version INT
);

INSERT INTO orders (order_id, customer_id, order_date, amount, version) VALUES
(1, 1001, '2023-01-01', 50.00, 1),
(2, 1001, '2023-02-01', 55.00, 2),
(3, 1001, '2023-03-01', 60.00, 3),
(4, 1002, '2023-01-01', 70.00, 1),
(5, 1002, '2023-02-01', 75.00, 2);
```

**Flow:**

1. Define Window Function
2. Check Order By Key
3. Missing Versioning Key
4. Add Version to ORDER BY
5. Re-run Query and ValidateResults

