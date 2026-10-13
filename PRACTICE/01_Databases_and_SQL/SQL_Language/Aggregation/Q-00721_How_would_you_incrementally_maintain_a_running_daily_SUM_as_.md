---
type: "practice"
difficulty: "Deep"
---

# How would you incrementally maintain a running daily SUM as new rows arrive, without rescanning the full fact table each night?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Advanced Aggregation](<../../../../01_Databases_and_SQL/SQL_Language/Aggregation/Advanced_Aggregation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the problem of incrementally maintaining a running daily sum as new rows arrive, we need to ensure that our database system can efficiently update this aggregate function without resorting to rescanning the entire fact table each night. This requires careful consideration of both query performance and data storage strategies.

The core idea is to leverage incremental updates through materialized views or streams. We will break down the solution from first principles, detailing how we implement this using a database stream and a materialized view.

**Code:**

```sql
-- Example schema for a simple fact table
CREATE TABLE sales (
    id INT PRIMARY KEY,
    product_id INT,
    sale_date DATE,
    amount DECIMAL(10, 2)
);

-- Create a materialized view to store the daily running sum
CREATE MATERIALIZED VIEW daily_sales_sum AS
SELECT
    sale_date,
    SUM(amount) OVER (ORDER BY sale_date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) as daily_running_total
FROM sales;

-- Ensure proper indexing and setup for efficient streaming updates
CREATE INDEX idx_sales_sale_date ON sales(sale_date);
```

**Example input / output:**

```text
-- Example data
INSERT INTO sales (id, product_id, sale_date, amount) VALUES
(1, 101, '2023-10-01', 50.00),
(2, 102, '2023-10-02', 75.00),
(3, 103, '2023-10-03', 60.00);

-- Update the materialized view to reflect new data
REFRESH MATERIALIZED VIEW daily_sales_sum;

-- Query the running total for a specific date
SELECT * FROM daily_sales_sum WHERE sale_date = '2023-10-03';
```

