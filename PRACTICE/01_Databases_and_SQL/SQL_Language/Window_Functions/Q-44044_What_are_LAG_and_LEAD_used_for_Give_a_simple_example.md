---
type: "practice"
difficulty: "Basic"
---

# What are LAG() and LEAD() used for? Give a simple example.

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ LAG and LEAD](<../../../../01_Databases_and_SQL/SQL_Language/Window_Functions/LAG_and_LEAD.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Lag() and Lead() are window functions in SQL that allow you to access rows relative to the current row, without needing to join or subquery the table multiple times. They are incredibly useful for analyzing time series data, calculating differences between consecutive rows, and much more.

Essentially, LAG() returns the value of an expression evaluated at a preceding row (n rows before), while LEAD() does the same but for a following row (n rows after). Here’s how they work:

SQL
-- Example: Calculate the difference in sales between current month and previous month
SELECT
    sale_date,
    sales_amount,
    LAG(sales_amount) OVER (ORDER BY sale_date) AS prev_sales_amount,
    sales_amount - LAG(sales_amount) OVER (ORDER BY sale_date) AS diff_sales
FROM
    sales_data;

In this example, LAG(sales_amount) brings the value of sales_amount from the previous row relative to the current one in the order defined by sale_date.

**Comparison:**

| sale_date   | sales_amount | prev_sales_amount | diff_sales    |
|-------------|--------------|-------------------|---------------|
| 2023-10-01  | 500          | NULL              | NULL          |
| 2023-10-02  | 600          | 500               | 100           |
| 2023-10-03  | 700          | 600               | 100           |

**Code:**

```sql
-- Example SQL query using LAG()
SELECT
    sale_date,
    sales_amount,
    LAG(sales_amount) OVER (ORDER BY sale_date) AS prev_sales_amount,
    sales_amount - LAG(sales_amount) OVER (ORDER BY sale_date) AS diff_sales
FROM
    sales_data;
```

**Example input / output:**

```

Given the input data:

JSON
Copy
[
  {"sale_date": "2023-10-01", "sales_amount": 500},
  {"sale_date": "2023-10-02", "sales_amount": 600},
  {"sale_date": "2023-10-03", "sales_amount": 700}
]

The output would be:

SQL
Copy
+------------+-------------+--------------------+------------------+
| sale_date  | sales_amount| prev_sales_amount  | diff_sales       |
+------------+-------------+--------------------+------------------+
| 2023-10-01 | 500         | NULL               | NULL             |
| 2023-10-02 | 600         | 500                | 100              |
| 2023-10-03 | 700         | 600                | 100              |
+------------+-------------+--------------------+------------------+
```

**Data:**

```json
{
  "sales_data": [
    {"sale_date": "2023-10-01", "sales_amount": 500},
    {"sale_date": "2023-10-02", "sales_amount": 600},
    {"sale_date": "2023-10-03", "sales_amount": 700}
  ]
}
```

**Flow:**

1. Current Row
2. LAG(Previous Row)
3. Value of Previous Row
4. LEAD(Next Row)
5. Value of Next Row

