---
type: "practice"
difficulty: "Deep"
---

# How would you generate a dense date series (a row per day) to left-join sales onto, filling missing days with zero?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Date Arithmetic](<../../../../TOOLS/MySQL/MySQL_Platform/Date_Arithmetic.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To generate a dense date series in MySQL and left-join it with sales data, ensuring that all dates are included even if there's no corresponding sale, we need to follow a multi-step approach. This involves creating a sequence of dates, ensuring the sequence covers the entire range of interest, and then performing a LEFT JOIN between this date series and the actual sales data.

The key idea is to leverage MySQL’s windowing functions and variables to generate a dense set of dates efficiently. We’ll start by identifying the date range from the sales data, then create a sequential set of dates within that range, and finally perform the join with zero-filled values for missing days.

**Comparison:**

| Date       | Sales Amount |
|------------|--------------|
| 2023-09-01 | 50           |
| 2023-09-02 | 0            |
| 2023-09-03 | 0            |
| 2023-09-04 | 0            |
| 2023-09-05 | 75           |
| 2023-09-06 | 0            |
| 2023-09-07 | 0            |
| 2023-09-08 | 100          |
| 2023-09-09 | 0            |

**Code:**

```

-- Step 1: Identify the min and max dates in the sales table SET @min_date = (SELECT MIN(sales_date) FROM sales); SET @max_date = (SELECT MAX(sales_date) FROM sales);

-- Step 2: Create a dense date series using variables WITH RECURSIVE date_series AS ( SELECT @min_date as date UNION ALL SELECT DATE_ADD(date, INTERVAL 1 DAY) FROM date_series WHERE DATE_ADD(date, INTERVAL 1 DAY) <= @max_date )

-- Step 3: Perform the left join with sales data and fill missing values with zero SELECT ds.date, COALESCE(s.sales_amount, 0) as sales_amount FROM ( SELECT MIN(sales_date) - INTERVAL (DAY(MIN(sales_date)) - 1) DAY + INTERVAL 1 DAY as start_date, MAX(sales_date) + INTERVAL (31 - DAY(MAX(sales_date))) DAY - INTERVAL 1 DAY as end_date FROM sales ) sd JOIN date_series ds ON ds.date BETWEEN sd.start_date AND sd.end_date LEFT JOIN sales s ON DATE(s.sales_date) = ds.date;

CODE
Copy

```sql
```

**Example input / output:**

```text
-- Example Input
WITH sales_data AS (
    SELECT * FROM JSON_TABLE(
        '[{"sales_id": 1, "sales_amount": 50, "sales_date": "2023-09-01"}, {"sales_id": 2, "sales_amount": 75, "sales_date": "2023-09-05"}, {"sales_id": 3, "sales_amount": 100, "sales_date": "2023-09-08"}]'
        COLUMNS (
            sales_id INT PATH '$.sales_id',
            sales_amount DECIMAL(10, 2) PATH '$.sales_amount',
            sales_date DATE PATH '$.sales_date'
        )
    ) AS jt
)

-- Example Output
WITH RECURSIVE date_series AS (
    SELECT MIN(sales_date) as date FROM sales_data
    UNION ALL
    SELECT DATE_ADD(date, INTERVAL 1 DAY)
    FROM date_series
    WHERE DATE_ADD(date, INTERVAL 1 DAY) <= (SELECT MAX(sales_date) + INTERVAL (31 - DAY(MAX(sales_date))) DAY - INTERVAL 1 DAY FROM sales_data)
)

SELECT ds.date, COALESCE(sd.sales_amount, 0) as sales_amount
FROM (
    SELECT MIN(sales_date) - INTERVAL (DAY(MIN(sales_date)) - 1) DAY + INTERVAL 1 DAY as start_date,
           MAX(sales_date) + INTERVAL (31 - DAY(MAX(sales_date))) DAY - INTERVAL 1 DAY as end_date
    FROM sales_data
) sd
JOIN date_series ds ON ds.date BETWEEN sd.start_date AND sd.end_date
LEFT JOIN sales_data sd ON DATE(sd.sales_date) = ds.date;
```

**Data:**

```json
{
  "sales": [
    {"sales_id": 1, "sales_amount": 50, "sales_date": '2023-09-01'},
    {"sales_id": 2, "sales_amount": 75, "sales_date": '2023-09-05'},
    {"sales_id": 3, "sales_amount": 100, "sales_date": '2023-09-08'}
  ]
}
```

**Flow:**

1. Generate Min and Max Dates
2. Create Date Series
3. Perform Left Join with Zero Fill
4. Output Final Data

