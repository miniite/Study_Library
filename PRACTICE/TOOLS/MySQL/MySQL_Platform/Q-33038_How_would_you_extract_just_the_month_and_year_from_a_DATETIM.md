---
type: "practice"
difficulty: "Basic"
---

# How would you extract just the month and year from a DATETIME and group sales by month?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Date Functions and Grouping](<../../../../TOOLS/MySQL/MySQL_Platform/Date_Functions_and_Grouping.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To extract just the month and year from a DATETIME column in MySQL and group sales by these dates, we need to use SQL functions to manipulate and aggregate data effectively. This problem tests our understanding of date functions and grouping queries in MySQL.

#### Triaging the Question

The question is asking how to format a DATETIME field to just show the month and year part (e.g., '2023-10' for October 2023) and then group sales data by this formatted date. The key functions to use here are DATE_FORMAT() for formatting dates and GROUP BY for aggregation.

#### First Principles Solution

We'll start with a basic query that extracts the month and year from the DATETIME field using DATE_FORMAT(). Then, we'll group our sales data by this extracted date. Here’s how you can achieve it:

Extract Month and Year: Use DATE_FORMAT() to format the DATETIME column.
Group By: Group the results by the formatted month and year.
Full Query

Here is a sample query in MySQL:

SQL
SELECT
    DATE_FORMAT(sales_datetime, '%Y-%m') AS formatted_date,
    SUM(sales_amount) AS total_sales
FROM
    sales_table
GROUP BY
    DATE_FORMAT(sales_datetime, '%Y-%m');
Explanation
DATE_FORMAT(sales_datetime, '%Y-%m'): This function extracts the year and month from the sales_datetime column and formats it as a string in the format 'YYYY-MM'.
SUM(sales_amount) AS total_sales: This calculates the total sales for each group of formatted dates.
GROUP BY DATE_FORMAT(sales_datetime, '%Y-%m'): This groups the results by the month and year extracted from the sales_datetime column.

#### Closing Summary

In summary, we used DATE_FORMAT() to extract the necessary date information from a DATETIME field and then grouped our sales data using this formatted date. The key is to use the correct SQL functions for formatting dates and grouping results.

#### Common Mistakes/Follow-Ups
Mistake: Forgetting to include the AS keyword when aliasing the column name.

#### Follow-Up
If there are multiple columns in the sales table, you might need to aggregate those as well. For example:
SQL
SELECT
    DATE_FORMAT(sales_datetime, '%Y-%m') AS formatted_date,
    SUM(sales_amount) AS total_sales,
    AVG(sales_price) AS average_price
FROM
    sales_table
GROUP BY
    DATE_FORMAT(sales_datetime, '%Y-%m');

This query also includes an additional aggregation function AVG() to calculate the average price per month.

**Code:**

```sql
SELECT
    DATE_FORMAT(sales_datetime, '%Y-%m') AS formatted_date,
    SUM(sales_amount) AS total_sales
FROM
    sales_table
GROUP BY
    DATE_FORMAT(sales_datetime, '%Y-%m');
```

