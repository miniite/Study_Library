---
type: "practice"
difficulty: "Intermediate"
---

# How do you write a query to fill gaps in a daily sales series so days with zero sales still appear, using a calendar table?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Expanding Date Ranges](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Expanding_Date_Ranges.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the need of filling gaps in a daily sales series where days without any sales should still be represented, we can leverage SQL to join a calendar table with the existing sales data and then use appropriate aggregation methods to ensure all dates are present. The core idea is to create a complete date range using the calendar table and left-join it with the original sales data to fill in missing values.

**Comparison:**

| date       | total_sales  |
|------------|--------------|
| 2023-10-01 | 50           |
| 2023-10-02 | 0            |
| 2023-10-03 | 75           |
| 2023-10-04 | 0            |
| 2023-10-05 | 100          |

**Code:**

```sql
-- Assuming we have two tables:
-- `sales` table with columns `sale_date` and `total_sales`
-- `calendar` table with a single column `date`

WITH complete_dates AS (
    SELECT date FROM calendar
)
SELECT cd.date, COALESCE(s.total_sales, 0) as total_sales
FROM complete_dates cd
LEFT JOIN sales s ON cd.date = s.sale_date;
```

**Example input / output:**

```text
Input:
Sales Data (sales table):
+------------+--------------+
| sale_date  | total_sales  |
+------------+--------------+
| 2023-10-01 | 50           |
| 2023-10-03 | 75           |
| 2023-10-05 | 100          |
+------------+--------------+

Calendar Data (calendar table):
2023-10-01
2023-10-02
2023-10-03
2023-10-04
2023-10-05

Output:
+------------+--------------+
| date       | total_sales  |
+------------+--------------+
| 2023-10-01 | 50           |
| 2023-10-02 | 0            |
| 2023-10-03 | 75           |
| 2023-10-04 | 0            |
| 2023-10-05 | 100          |
+------------+--------------+
```

**Data:**

```json
{
  "sales": [
    {"sale_date": "2023-10-01", "total_sales": 50},
    {"sale_date": "2023-10-03", "total_sales": 75},
    {"sale_date": "2023-10-05", "total_sales": 100}
  ],
  "calendar": [
    "2023-10-01",
    "2023-10-02",
    "2023-10-03",
    "2023-10-04",
    "2023-10-05"
  ]
}
```

**Flow:**

1. Yes
2. No
3. Complete Dates CTE
4. Does the date exist in sales?
5. Return total_sales from sales
6. Return 0 for missing days
7. Output total_sales
8. Left join with calendar andcomplete_dates CTE

