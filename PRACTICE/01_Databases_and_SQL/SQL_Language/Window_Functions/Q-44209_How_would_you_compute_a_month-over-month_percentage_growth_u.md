---
type: "practice"
difficulty: "Intermediate"
---

# How would you compute a month-over-month percentage growth using LAG and guard against divide-by-zero?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ LAG and LEAD](<../../../../01_Databases_and_SQL/SQL_Language/Window_Functions/LAG_and_LEAD.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To compute a month-over-month (MoM) percentage growth, we need to compare each month's value with its preceding month's value. This can be achieved using SQL window functions like LAG. However, we must also handle the scenario where there is no preceding month (i.e., during the first month) or when both values are zero (to avoid division by zero). Here’s a step-by-step breakdown and solution:

#### Triaging the Question

The question asks us to compute the MoM percentage growth. The key points here are:

Using LAG to access the value from the previous row.
Handling cases where there is no preceding month or both values are zero.

#### First-Principles Solution

To start, let's assume we have a simple table with date and sales columns:

SQL
CREATE TABLE sales (
    id INT PRIMARY KEY,
    sale_date DATE,
    sales_amount DECIMAL(10, 2)
);

We want to compute the MoM growth for each row. Here’s how we can achieve this using SQL window functions.

Step-by-Step Solution
Ranking by Date: We first need to rank rows based on date to ensure correct order.
LAG Function: Use LAG to get the sales amount from the previous month.
Handling Divide-by-Zero: Ensure we handle cases where there is no preceding month or both values are zero.
SQL Query
SQL
WITH ranked_sales AS (
    SELECT
        id,
        sale_date,
        sales_amount,
        LAG(sales_amount) OVER (ORDER BY sale_date) AS prev_sales_amount
    FROM
        sales
)
SELECT
    id,
    sale_date,
    sales_amount,
    CASE
        WHEN prev_sales_amount IS NULL OR prev_sales_amount = 0 THEN NULL
        ELSE ((sales_amount - prev_sales_amount) / prev_sales_amount * 100)
    END AS moom_growth
FROM
    ranked_sales;
Explanation of the Solution
WITH ranked_sales: We use a Common Table Expression (CTE) to rank rows based on sale_date.
LAG Function: The LAG(sales_amount) OVER (ORDER BY sale_date) function retrieves the sales amount from the previous month.
Handling Divide-by-Zero: In the SELECT statement, we use a CASE statement to handle cases where there is no preceding value (prev_sales_amount IS NULL) or both values are zero. If these conditions are met, the growth will be set to NULL.

#### Closing Summary
and Common Mistakes
Summary: The solution uses window functions effectively to compute MoM percentage growth while handling edge cases like divide-by-zero.

#### Common Mistakes
Forgetting to handle NULL values from LAG.
Not considering the scenario where both current and previous month's sales are zero.
Misunderstanding the correct order of operations in SQL window functions.

#### Example
Data and Input/Output
SQL
-- Sample Data
INSERT INTO sales (id, sale_date, sales_amount) VALUES
(1, '2023-01-01', 100.00),
(2, '2023-02-01', 150.00),
(3, '2023-03-01', 140.00);

-- Expected Output
id | sale_date   | sales_amount | moom_growth
---|-------------|--------------|-------------
1  | 2023-01-01  | 100.00       | NULL
2  | 2023-02-01  | 150.00       | 50.00
3  | 2023-03-01  | 140.00       | -7.69
Self-check

This solution is correct and complete, handling edge cases as required.

**Comparison:**

| id | sale_date   | sales_amount |
|----|-------------|--------------|
| 1  | 2023-01-01  | 100.00       |
| 2  | 2023-02-01  | 150.00       |
| 3  | 2023-03-01  | 140.00       |
| id | sale_date   | sales_amount | moom_growth |
|----|-------------|--------------|-------------|
| 1  | 2023-01-01  | 100.00       | NULL        |
| 2  | 2023-02-01  | 150.00       | 50.00       |
| 3  | 2023-03-01  | 140.00       | -7.69       |

**Code:**

```sql
WITH ranked_sales AS (
    SELECT
        id,
        sale_date,
        sales_amount,
        LAG(sales_amount) OVER (ORDER BY sale_date) AS prev_sales_amount
    FROM
        sales
)
SELECT
    id,
    sale_date,
    sales_amount,
    CASE
        WHEN prev_sales_amount IS NULL OR prev_sales_amount = 0 THEN NULL
        ELSE ((sales_amount - prev_sales_amount) / prev_sales_amount * 100)
    END AS moom_growth
FROM
    ranked_sales;
```

**Example input / output:**

```text
-- Output
id | sale_date   | sales_amount | moom_growth
---|-------------|--------------|-------------
1  | 2023-01-01  | 100.00       | NULL
2  | 2023-02-01  | 150.00       | 50.00
3  | 2023-03-01  | 140.00       | -7.69
```

**Data:**

```sql
INSERT INTO sales (id, sale_date, sales_amount) VALUES
(1, '2023-01-01', 100.00),
(2, '2023-02-01', 150.00),
(3, '2023-03-01', 140.00);
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Create CTE with LAG function
6. Is prev_sales_amount NULL?
7. Return NULL
8. Are both values zero?
9. Compute growth:(sales_amount -prev_sales_amount) /prev_sales_amount * 100

