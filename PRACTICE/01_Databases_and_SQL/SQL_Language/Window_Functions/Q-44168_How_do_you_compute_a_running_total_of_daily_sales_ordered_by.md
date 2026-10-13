---
type: "practice"
difficulty: "Basic"
---

# How do you compute a running total of daily sales ordered by date? Write the query.

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ First Last per Group and Running Totals](<../../../../01_Databases_and_SQL/SQL_Language/Window_Functions/First_Last_per_Group_and_Running_Totals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To compute a running total of daily sales ordered by date, we use SQL window functions to maintain an ongoing sum as rows are processed in order. The SUM() function combined with the OVER() clause is key here.

**Code:**

```sql
-- Example query for computing running total sales
WITH daily_sales AS (
    SELECT
        sale_date,
        amount,
        SUM(amount) OVER (ORDER BY sale_date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) as running_total
    FROM
        sales_table
)
SELECT * FROM daily_sales;
```

**Example input / output:**

```text
-- Example input data
WITH sales_table (sale_date, amount) AS (
    VALUES ('2023-10-01', 500),
           ('2023-10-02', 700),
           ('2023-10-03', 900),
           ('2023-10-04', 800)
)

-- Query result
sale_date | amount | running_total
-----------|--------|--------------
2023-10-01 |   500  |         500
2023-10-02 |   700  |        1200
2023-10-03 |   900  |        2100
2023-10-04 |   800  |        2900
```

