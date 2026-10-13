---
type: "practice"
difficulty: "Basic"
---

# What is the difference between putting SUM() OVER () and just SUM() with a GROUP BY in terms of how many rows you get back?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Ordering Inside and Outside OVER](<../../../../01_Databases_and_SQL/SQL_Language/Window_Functions/Ordering_Inside_and_Outside_OVER.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In SQL, SUM() without any window function behaves like a typical aggregation function, reducing the result set to fewer rows by grouping and summing values. In contrast, SUM() OVER () with a window function retains all the original rows in the output but calculates the cumulative or partitioned sum for each row. Let's break down both scenarios step by step.

#### Triage
Understanding the Question

The core of this question is to understand how aggregation functions behave differently when used without a GROUP BY clause and when they are part of a window function. Specifically, we need to compare:

How many rows are returned in each case.
The nature of the aggregated values.
First Principles: Aggregation Basics
Aggregation with Group By: This method groups data based on one or more columns (or expressions) and then applies an aggregation function like SUM() to those groups. The result is a summarized view where each row represents a group, and there are usually fewer rows in the output.
Window Function Aggregation: Here, the aggregation function (SUM()) operates over a window of rows within the partition, but the original number of rows remains unchanged. Each row still retains its individual identity.
Step-by-Step Solution

Let's use a simple example to illustrate these differences:

#### Example
Schema
SQL
CREATE TABLE sales (
    order_id INT,
    product_name VARCHAR(50),
    quantity INT,
    price DECIMAL(10, 2)
);
Sample Data
SQL
INSERT INTO sales (order_id, product_name, quantity, price) VALUES
(1, 'Widget A', 10, 1.50),
(1, 'Widget B', 20, 2.00),
(2, 'Gadget X', 30, 3.50),
(2, 'Gadget Y', 40, 4.00);

#### Example
Query without GROUP BY

This query calculates the total quantity of each product across all orders:

SQL
SELECT
    product_name,
    SUM(quantity) AS total_quantity
FROM
    sales
GROUP BY
    product_name;
Output: This will return 2 rows, one for 'Widget A' and another for 'Gadget X'.

#### Example
Query with Window Function

This query calculates the cumulative sum of quantity for each order:

SQL
SELECT
    order_id,
    product_name,
    quantity,
    SUM(quantity) OVER (PARTITION BY order_id) AS cumulative_quantity
FROM
    sales;
Output: This will return 4 rows, with an additional column showing the cumulative quantity per order_id.

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary:

Using SUM() without a GROUP BY reduces the number of rows by summing across groups.
Incorporating SUM() into a window function retains all original rows, but still allows for row-wise aggregation.

#### Common mistakes
to avoid:

Assuming that adding a window function automatically changes the number of returned rows.
Misunderstanding the purpose and behavior of both GROUP BY and window functions.

#### Follow-up
questions could involve scenarios where you need to combine these techniques or understand their performance implications in large datasets.

**Comparison:**

| Without GROUP BY Results |
|--------------------------|
| product_name | total_quantity |
| Widget A     | 10            |
| Gadget X     | 30            |
| With Window Function Results |
|------------------------------|
| order_id   | product_name  | quantity | cumulative_quantity |
|------------|---------------|---------|--------------------|
| 1          | Widget A      | 10      | 20                 |
| 1          | Widget B      | 20      | 20                 |
| 2          | Gadget X      | 30      | 70                 |
| 2          | Gadget Y      | 40      | 70                 |

**Code:**

```sql
-- Example query without GROUP BY
SELECT
    product_name,
    SUM(quantity) AS total_quantity
FROM
    sales
GROUP BY
    product_name;

-- Example query with window function
SELECT
    order_id,
    product_name,
    quantity,
    SUM(quantity) OVER (PARTITION BY order_id) AS cumulative_quantity
FROM
    sales;
```

**Example input / output:**

```text
Without GROUP BY:
+-----------------------------+
| product_name | total_quantity |
+-----------------------------+
| Widget A     | 10            |
+-----------------------------+
| Gadget X     | 30            |
+-----------------------------+

With window function:
+------------+---------------+--------+--------------------+
| order_id   | product_name  | quantity | cumulative_quantity |
+------------+---------------+---------+--------------------+
| 1          | Widget A      | 10      | 20                 |
+------------+---------------+---------+--------------------+
| 1          | Widget B      | 20      | 20                 |
+------------+---------------+---------+--------------------+
| 2          | Gadget X      | 30      | 70                 |
+------------+---------------+---------+--------------------+
| 2          | Gadget Y      | 40      | 70                 |
+------------+---------------+---------+--------------------+
```

**Data:**

```sql
CREATE TABLE sales (
    order_id INT,
    product_name VARCHAR(50),
    quantity INT,
    price DECIMAL(10, 2)
);

INSERT INTO sales (order_id, product_name, quantity, price) VALUES
(1, 'Widget A', 10, 1.50),
(1, 'Widget B', 20, 2.00),
(2, 'Gadget X', 30, 3.50),
(2, 'Gadget Y', 40, 4.00);
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Input Table
6. SUM() without GROUP BY
7. Rows?
8. Reduced Rows per Group
9. All Rows retained
10. Input Table
11. SUM() OVER () WindowFunction
12. Rows?
13. All Rows with AggregatedValues
14. Unexpected Output

