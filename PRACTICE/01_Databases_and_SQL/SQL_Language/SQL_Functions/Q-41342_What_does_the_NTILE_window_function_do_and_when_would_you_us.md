---
type: "practice"
difficulty: "Deep"
---

# What does the NTILE window function do and when would you use it to create buckets?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Window Functions Overview](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Functions/Window_Functions_Overview.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

The NTILE window function in SQL is used to partition a set of rows into a specified number of groups (buckets) that are as evenly distributed as possible. This can be very useful when you need to categorize data into quantiles or apply some kind of ranking without using an aggregate function.

#### Triage

The question asks for the definition and use case of the NTILE window function. The interviewer is probing your understanding of SQL window functions, particularly how they differ from aggregate functions like COUNT, SUM, etc., and how you can leverage them to categorize data in a non-aggregated manner.

#### First-Principles Solution
Definition: The NTILE(n) function divides the result set into a specified number of buckets (n). Each bucket will have approximately an equal number of rows, with any remainder distributed one row per bucket.
Syntax: SELECT NTILE(num_of_buckets) OVER (PARTITION BY column_name ORDER BY another_column) AS bucket_column FROM table_name;
Use Cases:
Quantile Analysis: To categorize data into quantiles or deciles, which is useful in statistical analysis.
Load Balancing: Distributing rows across multiple partitions for load balancing purposes.
Segmentation: Grouping users or transactions based on certain criteria.

#### Example

Let's say we have a table of sales data with columns product_id, sale_date, and amount. We want to categorize the products into 4 buckets based on their total sales.

SQL
SELECT
    product_id,
    sale_date,
    amount,
    NTILE(4) OVER (PARTITION BY product_id ORDER BY amount DESC) AS sales_bucket
FROM
    sales_data;

In this query:

We partition the data by product_id to ensure that each product is grouped separately.
We order by amount in descending order so that higher amounts get lower buckets, effectively creating quartiles.

#### Common Mistakes
and Follow-Ups
Misunderstanding Order by Clause: Forgetting or incorrectly using the ORDER BY clause can lead to incorrect bucketing.
Incorrect Number of Buckets: Using a number of buckets that doesn't make sense for your data, leading to less meaningful results.
Ignoring Partitioning: Not properly partitioning the data can result in a different distribution than intended.

#### Closing Summary

The NTILE function is a powerful tool for categorizing rows into evenly distributed groups. It's particularly useful when you need to perform quantile analysis or load balancing. Understanding how and when to use NTILE will greatly enhance your SQL skills, especially in scenarios where data needs to be categorized without aggregation.

**Comparison:**

```
product_id	sale_date	amount
101	2023-10-01	500
101	2023-10-02	450
101	2023-10-03	380
102	2023-10-01	600
102	2023-10-02	550
102	2023-10-03	480

The output will look like this:

product_id	sale_date	amount	sales_bucket
101	2023-10-01	500	1
101	2023-10-02	450	2
101	2023-10-03	380	3
102	2023-10-01	600	1
102	2023-10-02	550	2
102	2023-10-03	480	3

In this example, the products are categorized into three buckets based on their sales amounts.
```

**Code:**

```sql
-- Example SQL code
SELECT
    product_id,
    sale_date,
    amount,
    NTILE(4) OVER (PARTITION BY product_id ORDER BY amount DESC) AS sales_bucket
FROM
    sales_data;
```

