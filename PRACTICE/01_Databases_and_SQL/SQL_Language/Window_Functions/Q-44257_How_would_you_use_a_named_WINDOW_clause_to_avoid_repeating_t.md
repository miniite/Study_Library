---
type: "practice"
difficulty: "Intermediate"
---

# How would you use a named WINDOW clause to avoid repeating the same OVER definition across three window columns?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Ordering Inside and Outside OVER](<../../../../01_Databases_and_SQL/SQL_Language/Window_Functions/Ordering_Inside_and_Outside_OVER.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we are tasked with using named window clauses in SQL to avoid duplicating the OVER clause when defining multiple window functions. This not only improves readability but also makes maintenance easier. Let's break down the solution step by step.

#### Triaging the Question

The core of the question is about reducing redundancy in SQL queries involving window functions. Specifically, we need a way to define the same window specification (e.g., OVER (PARTITION BY ...)) once and reuse it across multiple columns or expressions.

#### Assumptions
We are working with a relational database that supports window functions.
The data is partitioned by one or more columns, and we want to perform calculations over these partitions.

#### First Principles Solution

To achieve this, we can use a named window clause. Here's how it works:

Define the Window: Create a named window using WINDOW keyword.
Use Named Window: Reference the named window in multiple window functions without redefining it each time.

Let's walk through an example to illustrate this concept.

#### Example

Suppose we have a sales dataset and we want to calculate the total sales and average sales for each product category, while also ranking products within each category based on their sales. The data looks like this:

ProductID	Category	SalesAmount
1	A	200
2	B	350
3	A	400
4	B	250

We want to calculate:

The total sales for each category.
The average sales per product within each category.
The rank of products based on their sales within each category.
Step-by-Step Solution
Define the Window:
SQL
WITH SalesData AS (
    SELECT
        ProductID,
        Category,
        SalesAmount
    FROM
        sales_table
),
WindowSpecs AS (
    SELECT
        *,
        SUM(SalesAmount) OVER (PARTITION BY Category) AS TotalSalesCategory,
        AVG(SalesAmount) OVER (PARTITION BY Category) AS AvgSalesPerProduct
    FROM
        SalesData
)
SELECT
    ProductID,
    Category,
    SalesAmount,
    TotalSalesCategory,
    AvgSalesPerProduct,
    RANK() OVER (PARTITION BY Category ORDER BY SalesAmount DESC) AS RankBySales
FROM
    WindowSpecs;
Explanation:
In the WindowSpecs CTE, we define a named window called TotalSalesCategory and AvgSalesPerProduct.
We then use these named windows in subsequent calculations.

#### Closing Summary

Using named windows is an excellent way to reduce redundancy in SQL queries involving window functions. By defining the window once and reusing it, you enhance code readability and maintainability. This approach is particularly useful when multiple calculations are needed over the same partitioned data.

#### Common Mistakes
and Follow-Ups
Mistake: Overlooking the PARTITION BY clause in the named window definition.

#### Follow-Up
Discussing scenarios where different window specifications might be necessary even within a single query, such as cumulative sums or more complex aggregations.

By leveraging named windows, we can write cleaner and more efficient SQL code that is easier to understand and maintain.

**Comparison:**

| ProductID | Category | SalesAmount | TotalSalesCategory | AvgSalesPerProduct | RankBySales |
|-----------|----------|-------------|--------------------|-------------------|-------------|
| 1         | A        | 200         | 600                | 200               | 2           |
| 3         | A        | 400         | 600                | 200               | 1           |
| 2         | B        | 350         | 600                | 300               | 1           |
| 4         | B        | 250         | 600                | 300               | 2           |

**Code:**

```sql
WITH SalesData AS (
    SELECT
        ProductID,
        Category,
        SalesAmount
    FROM
        sales_table
),
WindowSpecs AS (
    SELECT
        *,
        SUM(SalesAmount) OVER (PARTITION BY Category) AS TotalSalesCategory,
        AVG(SalesAmount) OVER (PARTITION BY Category) AS AvgSalesPerProduct
    FROM
        SalesData
)
SELECT
    ProductID,
    Category,
    SalesAmount,
    TotalSalesCategory,
    AvgSalesPerProduct,
    RANK() OVER (PARTITION BY Category ORDER BY SalesAmount DESC) AS RankBySales
FROM
    WindowSpecs;
```

**Example input / output:**

```sql
WITH SalesData AS (
    SELECT
        ProductID,
        Category,
        SalesAmount
    FROM
        sales_table
),
WindowSpecs AS (
    SELECT
        *,
        SUM(SalesAmount) OVER (PARTITION BY Category) AS TotalSalesCategory,
        AVG(SalesAmount) OVER (PARTITION BY Category) AS AvgSalesPerProduct
    FROM
        SalesData
)
SELECT
    ProductID,
    Category,
    SalesAmount,
    TotalSalesCategory,
    AvgSalesPerProduct,
    RANK() OVER (PARTITION BY Category ORDER BY SalesAmount DESC) AS RankBySales
FROM
    WindowSpecs;
ProductID	Category	SalesAmount	TotalSalesCategory	AvgSalesPerProduct	RankBySales
3	A	400	600	200	1
1	A	200	600	200	2
2	B	350	600	300	1
4	B	250	600	300	2
```

**Data:**

```sql
CREATE TABLE sales_table (
    ProductID INT,
    Category VARCHAR(255),
    SalesAmount DECIMAL
);

INSERT INTO sales_table VALUES
(1, 'A', 200),
(2, 'B', 350),
(3, 'A', 400),
(4, 'B', 250);
```

**Flow:**

1. Partition by Category
2. Partition by Category
3. SalesData CTE
4. WindowSpecs CTE
5. Window Specification
6. SUM(SalesAmount) ASTotalSalesCategory
7. AVG(SalesAmount) ASAvgSalesPerProduct
8. RANK() BY SalesAmount DESC
9. TotalSalesCategory
10. AvgSalesPerProduct
11. RankBySales

