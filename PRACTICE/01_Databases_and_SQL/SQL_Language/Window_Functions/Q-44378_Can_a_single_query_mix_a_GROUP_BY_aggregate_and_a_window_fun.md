---
type: "practice"
difficulty: "Basic"
---

# Can a single query mix a GROUP BY aggregate and a window function together? Explain what each operates on.

**Difficulty:** Basic · **Levels:** Fresher · [↑ Window Function Fundamentals and Performance](<../../../../01_Databases_and_SQL/SQL_Language/Window_Functions/Window_Function_Fundamentals_and_Performance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

Yes, you can use both GROUP BY aggregates and window functions in a single SQL query. The key is to understand that GROUP BY works with sets of rows and applies aggregate operations to them, while window functions operate on the partitioned data within the result set.

#### Triage

The question asks whether it's possible to combine GROUP BY aggregates and window functions in a single query and what each operation targets. We need to clarify that both serve different purposes: GROUP BY groups rows together for aggregate calculations, whereas window functions operate on subsets of the result set defined by partitions.

Detailed Solution

Let's break down how these two operations work together:

Understanding GROUP BY:

The GROUP BY clause is used to partition a table into groups based on one or more columns.
For each group, an aggregate function (like COUNT, SUM, AVG, etc.) can be applied.

Introduction to Window Functions:

Window functions are designed to perform calculations across a set of rows that are related to the current row within a partition.
They provide more flexibility by operating on subsets of data, which can be defined using partitions and ordering clauses.
Combining GROUP BY and Window Functions

To illustrate this, consider an example where we want to analyze sales data for different regions while also computing rolling averages over time.

SQL
SELECT
    region,
    date,
    SUM(sales) AS total_sales,
    AVG(CASE WHEN date BETWEEN (date - INTERVAL '10 days') AND date THEN sales END) OVER (PARTITION BY region ORDER BY date) AS rolling_avg_sales
FROM
    sales_data
GROUP BY
    region, date
ORDER BY
    region, date;

In this query:

The SUM(sales) is a GROUP BY aggregate that calculates the total sales for each region and date.
The AVG() window function computes a rolling average of sales over the last 10 days within each partition (region) for each day.
Step-by-Step Explanation

Partitioning by Region:

We start with the GROUP BY clause, which groups the data by region and date.

Aggregating Total Sales:

For each combination of region and date, we calculate the total sales using SUM(sales).

Defining Rolling Windows:

The window function AVG() is defined within a OVER (PARTITION BY region ORDER BY date) clause.
This partitions the data by region and orders it by date to compute rolling averages over 10-day windows.

Result Set Construction:

Each row in the result set contains both the total sales for that region and date, as well as a rolling average of sales over the past 10 days.

#### Closing Summary

In summary, you can indeed mix GROUP BY aggregates with window functions in a single SQL query. The key is to recognize that:

GROUP BY operates on sets of rows grouped by one or more columns.
Window functions operate on subsets of data defined by partitions and ordering within those partitions.

This combination allows for powerful and flexible data analysis, enabling complex calculations like rolling averages alongside aggregate totals in a single query.

#### Common Mistakes/Follow-Ups

Misunderstanding GROUP BY:

Newcomers might confuse the use of GROUP BY with window functions.

Incorrect Window Partitioning:

Failing to define partitions correctly can lead to incorrect results or performance issues.

Overlooking ORDER BY in Window Functions:

For accurate rolling calculations, ensure the correct ordering is applied within the window function's partition.

Combining Multiple Aggregates with Window Functions:

Mixing multiple aggregate functions and window functions can be complex; it’s important to ensure clarity in the query logic.

These points highlight common pitfalls that developers should consider when writing such queries.

**Comparison:**

| region | date       | total_sales | rolling_avg_sales   |
|--------|------------|-------------|---------------------|
| North  | 2023-10-01 |      500.0  |                    NaN |
| North  | 2023-10-02 |     1100.0  |        600.000000000 |
| South  | 2023-10-01 |      400.0  |                    NaN |
| South  | 2023-10-02 |       900.0  |        500.000000000 |

**Code:**

```sql
SELECT
    region,
    date,
    SUM(sales) AS total_sales,
    AVG(CASE WHEN date BETWEEN (date - INTERVAL '10 days') AND date THEN sales END) OVER (PARTITION BY region ORDER BY date) AS rolling_avg_sales
FROM
    sales_data
GROUP BY
    region, date
ORDER BY
    region, date;
```

**Example input / output:**

```text
-- Input Query:
SELECT
    region,
    date,
    SUM(sales) AS total_sales,
    AVG(CASE WHEN date BETWEEN (date - INTERVAL '10 days') AND date THEN sales END) OVER (PARTITION BY region ORDER BY date) AS rolling_avg_sales
FROM
    sales_data
GROUP BY
    region, date
ORDER BY
    region, date;

-- Output:
+---------+------------+-------------+--------------------------------+
| region  |  date      | total_sales | rolling_avg_sales              |
+---------+------------+-------------+--------------------------------+
| North   | 2023-10-01 |        500.0 |                            NaN |
| North   | 2023-10-02 |       1100.0 |                  600.000000000 |
| South   | 2023-10-01 |        400.0 |                            NaN |
| South   | 2023-10-02 |         900.0 |                  500.000000000 |
+---------+------------+-------------+--------------------------------+
```

**Data:**

```json
{
  "sales_data": [
    {"region": "North", "date": "2023-10-01", "sales": 500},
    {"region": "North", "date": "2023-10-02", "sales": 600},
    {"region": "South", "date": "2023-10-01", "sales": 400},
    {"region": "South", "date": "2023-10-02", "sales": 500}
  ]
}
```

**Flow:**

1. Input Data
2. GROUP BY region, date
3. SUM(sales) as total_sales
4. AVG(CASE WHEN dateBETWEEN (date - INTERVAL '10days') AND date THEN salesEND) OVER (PARTITION BYregion ORDER BY date) asrolling_avg_sales
5. Output Data with total_salesand rolling_avg_sales

