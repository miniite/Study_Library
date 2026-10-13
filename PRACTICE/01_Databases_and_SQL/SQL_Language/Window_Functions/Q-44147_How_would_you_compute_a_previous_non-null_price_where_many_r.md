---
type: "practice"
difficulty: "Deep"
---

# How would you compute a 'previous non-null price' where many rows have null prices, using LAG with IGNORE NULLS or a framed last-value fallback?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ LAG and LEAD](<../../../../01_Databases_and_SQL/SQL_Language/Window_Functions/LAG_and_LEAD.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To compute a "previous non-null price" in SQL when dealing with many null values, we can use window functions like LAG combined with IGNORE NULLS, which is available in recent versions of databases such as PostgreSQL and SQL Server. Alternatively, we can employ a framed last-value fallback approach using Lag with appropriate frame specifications. This solution ensures that the first non-null price before each row is correctly identified.

**Comparison:**

```
FLOWCHART · MERMAID
gantt
    title Previous Non-null Price Computation
    section Using LAG with IGNORE NULLS
        2023-01-01 : Create table and insert sample data
        2023-01-05 : Write SQL query using `LAG` with `IGNORE NULLS`
    section Using Framed Last-value Fallback
        2023-01-06 : Write SQL query using framed last-value fallback
```

**Code:**

```sql
-- Example table creation
CREATE TABLE stock_prices (
    id SERIAL PRIMARY KEY,
    date DATE NOT NULL,
    price DECIMAL(10, 2)
);

-- Sample data insertion
INSERT INTO stock_prices (date, price) VALUES
('2023-01-01', 15.50),
('2023-01-02', NULL),
('2023-01-03', NULL),
('2023-01-04', 16.00),
('2023-01-05', NULL);

-- Using LAG with IGNORE NULLS
SELECT
    date,
    price,
    LAG(price, 1, NULL) OVER (ORDER BY date) FILTER (WHERE LAG(price, 1, NULL) IS NOT NULL) AS prev_non_null_price
FROM stock_prices;

-- Using framed last-value fallback
SELECT
    date,
    price,
    LAG(price, 1, (SELECT MAX(price) FROM stock_prices WHERE date < sp.date)) OVER (ORDER BY date) AS prev_non_null_price
FROM stock_prices;
```

**Example input / output:**

| date       | price | prev_non_null_price |
|------------|-------|---------------------|
| 2023-01-01 | 15.50 | NULL                |
| 2023-01-02 | NULL  | 15.50               |
| 2023-01-03 | NULL  | 15.50               |
| 2023-01-04 | 16.00 | NULL                |
| 2023-01-05 | NULL  | 16.00               |

**Data:**

```json
[
    {"id":1,"date":"2023-01-01","price":15.5},
    {"id":2,"date":"2023-01-02","price":null},
    {"id":3,"date":"2023-01-03","price":null},
    {"id":4,"date":"2023-01-04","price":16.0},
    {"id":5,"date":"2023-01-05","price":null}
]
```

**Flow:**

1. Yes
2. No
3. Start
4. Is database version >=required?
5. Use LAG with IGNORE NULLS
6. Use framed last-value fallback
7. Compute previous non-nullprice
8. Compute previous non-nullprice
9. End

