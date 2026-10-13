---
type: "practice"
difficulty: "Deep"
---

# Describe a production incident where a window function silently produced wrong results because the ORDER BY was not unique, and how you'd add a deterministic tiebreaker.

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Deterministic Deduplication](<../../../../01_Databases_and_SQL/SQL_Language/Window_Functions/Deterministic_Deduplication.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a critical production environment at a large e-commerce platform, we encountered an issue with a complex query involving window functions. The core problem arose because the ORDER BY clause in the query was not unique, leading to incorrect results without any immediate error messages or warnings. This incident highlighted the importance of ensuring deterministic ordering when using window functions.

**Comparison:**

```
customer_id	order_date	order_amount	order_id
1	2023-06-01	100	1
1	2023-06-01	50	2
1	2023-06-02	75	3
```

**Code:**

```sql
-- Example query causing issues
SELECT
    customer_id,
    order_date,
    SUM(order_amount) OVER (
        PARTITION BY customer_id
        ORDER BY order_date  -- Not unique, can lead to unexpected results
    ) as rolling_total
FROM orders;
```

**Example input / output:**

```text
-- Initial query (incorrect)
SELECT
    customer_id,
    order_date,
    SUM(order_amount) OVER (
        PARTITION BY customer_id
        ORDER BY order_date
    ) as rolling_total
FROM orders;

-- Incorrect results:
customer_id | order_date  | rolling_total
------------|-------------|--------------
1           | 2023-06-01  | 150          -- Should be 100, but incorrect due to duplicate date
1           | 2023-06-01  | 150          -- Should be 150, but incorrect due to duplicate date
1           | 2023-06-02  | 225          -- Correct, but rolling over with wrong starting point

-- Corrected query (with tiebreaker)
SELECT
    customer_id,
    order_date,
    SUM(order_amount) OVER (
        PARTITION BY customer_id
        ORDER BY order_date, order_id  -- Unique identifier added as tiebreaker
    ) as rolling_total
FROM orders;
```

**Data:**

```json
{
  "orders": [
    { "customer_id": 1, "order_date": "2023-06-01", "order_amount": 100 },
    { "customer_id": 1, "order_date": "2023-06-01", "order_amount": 50 },
    { "customer_id": 1, "order_date": "2023-06-02", "order_amount": 75 }
  ]
}
```

**Flow:**

1. No
2. Yes
3. Identify the Problem
4. Is ORDER BY unique?
5. Add deterministic tiebreaker
6. Query executes correctly
7. Modify query to includetiebreaker, e.g., order_date,order_id
8. Re-run query and verify results

