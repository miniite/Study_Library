---
type: "practice"
difficulty: "Deep"
---

# How would you compute year-over-year growth per category in one query, and why is a self-join to last year's grouped totals often the wrong tool versus LAG over grouped data?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Advanced Aggregation](<../../../../01_Databases_and_SQL/SQL_Language/Aggregation/Advanced_Aggregation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To compute year-over-year (YoY) growth per category in one query, we need to compare sales or metrics from two consecutive periods—typically from the current period and the same period of the previous year. The common approach is either using a self-join with grouped data or leveraging window functions like LAG. However, there are trade-offs between these methods.

In this scenario, I will explain why a self-join to last year's grouped totals might not be the best choice and demonstrate how to use LAG over grouped data instead.

**Comparison:**

| category    | cumulative_amount_2023 | prev_year_cumulative_amount_2022 |
|-------------|-----------------------:|--------------------------------:|
| Books       |                 85.00  |                               79.00 |
| Clothing    |                140.00   |                              130.00 |
| Electronics |               100.00   |                                 80.00 |

**Code:**

```

-- Sample Sales Data CREATE TABLE sales ( sale_id INT, category VARCHAR(50), sale_date DATE, amount DECIMAL(10, 2) );

INSERT INTO sales (sale_id, category, sale_date, amount) VALUES (1, 'Electronics', '2023-01-01', 100.00), (2, 'Books', '2023-01-05', 45.00), (3, 'Clothing', '2023-01-10', 75.00), -- Insert data for the previous year (11, 'Electronics', '2022-01-01', 80.00), (12, 'Books', '2022-01-05', 40.00), (13, 'Clothing', '2022-01-10', 65.00);

CODE
Copy

```sql
-- Using LAG to compute YoY growth
WITH category_sales AS (
    SELECT
        category,
        sale_date,
        amount
    FROM sales
)
SELECT
    category,
    SUM(amount) OVER (PARTITION BY category ORDER BY sale_date ROWS BETWEEN 1 PRECEDING AND CURRENT ROW) AS cumulative_amount,
    LAG(SUM(amount) OVER (PARTITION BY category ORDER BY sale_date))
      OVER (PARTITION BY category) * 1.0 AS prev_year_cumulative_amount
FROM category_sales
WHERE sale_date >= '2023-01-01' AND sale_date < '2024-01-01'
ORDER BY category, sale_date;
```

**Example input / output:**

| category | cumulative_amount | prev_year_cumulative_amount |
|----------|------------------:|---------------------------:|
| Books    |              85.00 |                        79.00 |
| Clothing |             140.00 |                       130.00 |
| Electronics |               100.00 |                          80.00 |

**Data:**

```sql
-- Sample Data Insertion
INSERT INTO sales (sale_id, category, sale_date, amount) VALUES
(1, 'Electronics', '2023-01-01', 100.00),
(2, 'Books', '2023-01-05', 45.00),
(3, 'Clothing', '2023-01-10', 75.00),
-- Insert data for the previous year
(11, 'Electronics', '2022-01-01', 80.00),
(12, 'Books', '2022-01-05', 40.00),
(13, 'Clothing', '2022-01-10', 65.00);
```

**Flow:**

1. Sales Data
2. Group by Category
3. Cumulative Sum
4. Lag Last Year's CumulativeAmount
5. YoY Growth Calculation

