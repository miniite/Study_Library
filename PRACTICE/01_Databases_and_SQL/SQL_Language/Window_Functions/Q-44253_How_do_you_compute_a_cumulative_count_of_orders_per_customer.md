---
type: "practice"
difficulty: "Intermediate"
---

# How do you compute a cumulative count of orders per customer that continues growing across dates (partitioned running COUNT)?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ First Last per Group and Running Totals](<../../../../01_Databases_and_SQL/SQL_Language/Window_Functions/First_Last_per_Group_and_Running_Totals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To compute a cumulative count of orders per customer that grows continuously over time, we can leverage window functions in SQL. Specifically, we'll use the OVER clause with the PARTITION BY and ORDER BY clauses to define the partitioning and ordering for our running total.

**Comparison:**

| customer_id | order_date | cumulative_order_count |
|-------------|------------|------------------------|
| 1           | 2023-01-01 | 1                      |
| 1           | 2023-01-05 | 2                      |
| 2           | 2023-01-03 | 1                      |
| 1           | 2023-01-08 | 3                      |
| 2           | 2023-01-07 | 2                      |

**Code:**

```sql
SELECT
    customer_id,
    order_date,
    COUNT(*) OVER (
        PARTITION BY customer_id
        ORDER BY order_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) as cumulative_order_count
FROM orders;
```

**Example input / output:**

```text
-- Input Data
INSERT INTO orders (customer_id, order_date) VALUES
(1, '2023-01-01'),
(1, '2023-01-05'),
(2, '2023-01-03'),
(1, '2023-01-08'),
(2, '2023-01-07');

-- Query
SELECT
    customer_id,
    order_date,
    COUNT(*) OVER (
        PARTITION BY customer_id
        ORDER BY order_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) as cumulative_order_count
FROM orders;

-- Output
+-------------+------------+-------------------+
| customer_id | order_date | cumulative_order_count |
+-------------+------------+-----------------------+
| 1           | 2023-01-01 | 1                     |
| 1           | 2023-01-05 | 2                     |
| 2           | 2023-01-03 | 1                     |
| 1           | 2023-01-08 | 3                     |
| 2           | 2023-01-07 | 2                     |
+-------------+------------+-----------------------+
```

**Data:**

| customer_id | order_date |
|-------------|------------|
| 1           | 2023-01-01 |
| 1           | 2023-01-05 |
| 2           | 2023-01-03 |
| 1           | 2023-01-08 |
| 2           | 2023-01-07 |

**Flow:**

1. Yes
2. Orders Table
3. Order Date < customer_id?
4. COUNT(*) per Customer
5. Row Number
6. RUNNING TOTAL

