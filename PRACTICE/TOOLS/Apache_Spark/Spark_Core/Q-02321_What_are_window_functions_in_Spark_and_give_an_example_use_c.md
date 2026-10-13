---
type: "practice"
difficulty: "Intermediate"
---

# What are window functions in Spark and give an example use case?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ DataFrames and Datasets](<../../../../TOOLS/Apache_Spark/Spark_Core/DataFrames_and_Datasets.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Window functions in Apache Spark allow you to perform aggregations over a sliding or fixed-size data window, which can be defined by partitioning the dataset based on certain columns and defining a frame of rows relative to each row. Essentially, it enables you to compute values across multiple rows that are related to the current row within a window.

#### Triaging the Question

The question is asking for an explanation of what window functions are in Apache Spark, and how they can be used practically. The interviewer might be looking to understand your grasp on the core concept, your ability to provide a relevant example, and your understanding of the syntax and usage patterns of window functions.

Key Assumptions
You have basic knowledge of SQL and distributed data processing.
You are familiar with Spark’s DataFrame/Dataset API.

#### First-Principles Solution

Window functions in Apache Spark operate on a sliding window or a frame of rows. These functions can be categorized into:

Row Number Window Functions: These provide the row number within a window, which is useful for ranking.
Aggregating Window Functions: These perform aggregations such as sum, average, max, etc., over a window.
Ranking and Densifying Window Functions: These rank rows based on certain conditions.

#### Example
Use Case: Calculating Moving Averages

A common use case for window functions is calculating moving averages or rolling aggregates. For instance, you might want to compute the average sales for each product over the past 7 days.

SQL
SELECT
    product_id,
    date,
    sales,
    AVG(sales) OVER (
        PARTITION BY product_id
        ORDER BY date
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) AS moving_avg_sales
FROM sales_table

In this example, PARTITION BY product_id ensures the window is defined per product. The ORDER BY date clause orders the rows in the window by date. The ROWS BETWEEN 6 PRECEDING AND CURRENT ROW specifies that the window includes the current row and the previous six rows.

#### Closing Summary

Window functions provide a powerful way to perform complex aggregations over a sliding or fixed-size data window, which is crucial for many real-world data analysis tasks such as calculating moving averages, cumulative totals, and rank-based computations. Understanding how to use these functions correctly can greatly enhance your ability to derive meaningful insights from large datasets.

#### Common Mistakes/Follow-Ups
Mistake: Forgetting the ORDER BY clause, which is essential for defining the window.
Mistake: Incorrectly setting the frame of rows (ROWS BETWEEN ...) leading to unexpected results.

#### Follow-Up
Discussing different types of window functions and their applications in various scenarios.

**Comparison:**

| product_id | date       | sales | moving_avg_sales |
|-----------|------------|-------|------------------|
| 1         | 2023-01-01 | 50    | NULL             |
| 1         | 2023-01-02 | 60    | 55               |
| 1         | 2023-01-03 | 70    | 65               |
| 1         | 2023-01-04 | 80    | 75               |
| 1         | 2023-01-05 | 90    | 85               |
| 1         | 2023-01-06 | 100   | 95               |
| 1         | 2023-01-07 | 110   | 105              |

**Code:**

```sql
SELECT
    product_id,
    date,
    sales,
    AVG(sales) OVER (
        PARTITION BY product_id
        ORDER BY date
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) AS moving_avg_sales
FROM sales_table
```

**Example input / output:**

```text
Input:
+---------+------------+-------+
| product_id | date       | sales |
+---------+------------+-------+
| 1        | 2023-01-01 | 50     |
| 1        | 2023-01-02 | 60     |
| 1        | 2023-01-03 | 70     |
| 1        | 2023-01-04 | 80     |
| 1        | 2023-01-05 | 90     |
| 1        | 2023-01-06 | 100    |
| 1        | 2023-01-07 | 110    |
+---------+------------+-------+

Output:
+---------+------------+-------+----------------------------------+
| product_id | date       | sales | moving_avg_sales                 |
+---------+------------+-------+----------------------------------+
| 1        | 2023-01-01 | 50     | NULL                             |
| 1        | 2023-01-02 | 60     | (50 + 60) / 2 = 55                |
| 1        | 2023-01-03 | 70     | (60 + 70) / 2 = 65                |
| 1        | 2023-01-04 | 80     | (70 + 80) / 2 = 75                |
| 1        | 2023-01-05 | 90     | (80 + 90) / 2 = 85                |
| 1        | 2023-01-06 | 100    | (90 + 100) / 2 = 95               |
| 1        | 2023-01-07 | 110    | (100 + 110) / 2 = 105             |
+---------+------------+-------+----------------------------------+
```

**Data:**

```json
{
  "sales_table": [
    {"product_id": 1, "date": "2023-01-01", "sales": 50},
    {"product_id": 1, "date": "2023-01-02", "sales": 60},
    {"product_id": 1, "date": "2023-01-03", "sales": 70},
    {"product_id": 1, "date": "2023-01-04", "sales": 80},
    {"product_id": 1, "date": "2023-01-05", "sales": 90},
    {"product_id": 1, "date": "2023-01-06", "sales": 100},
    {"product_id": 1, "date": "2023-01-07", "sales": 110}
  ]
}
```

**Flow:**

1. Sales Table
2. Order By Date
3. Partition by Product ID
4. Rows Between 6 Precedingand Current Row
5. Moving Avg Sales Calculation

