---
type: "practice"
difficulty: "Intermediate"
---

# What does the OVER() clause with PARTITION BY do in a window function?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ OR Conditions and Index Usage](<../../../../TOOLS/SQL_Server/SQL_Server_Platform/OR_Conditions_and_Index_Usage.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

The OVER() clause with PARTITION BY in SQL Server window functions is crucial for defining how rows are grouped and processed within the result set. Essentially, it allows you to perform calculations on subsets of data that share a common value or set of values, creating a partitioned view of your data. This feature is particularly useful when you want to compute running totals, averages, or other aggregations within specific groups.

To understand this better, let's break down the question and see how it works step by step.

**Code:**

```sql
-- Example table creation and population
CREATE TABLE Sales (
    SaleID INT,
    ProductID INT,
    Quantity INT,
    Price DECIMAL(10, 2),
    SaleDate DATE
);

INSERT INTO Sales (SaleID, ProductID, Quantity, Price, SaleDate)
VALUES
(1, 101, 5, 10.99, '2023-01-01'),
(2, 102, 7, 14.99, '2023-01-01'),
(3, 103, 8, 16.99, '2023-01-02'),
(4, 101, 5, 10.99, '2023-01-02'),
(5, 102, 7, 14.99, '2023-01-03');
```

**Example input / output:**

```sql
-- Basic example of using OVER() with PARTITION BY to calculate running totals per product
SELECT
    SaleID,
    ProductID,
    Quantity,
    Price,
    SUM(Quantity) OVER (PARTITION BY ProductID ORDER BY SaleDate ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS RunningTotal
FROM Sales;

The above query calculates the running total of Quantity for each ProductID, partitioned by ProductID. The ORDER BY SaleDate ensures that the rows are processed in chronological order, and ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW means that all preceding rows up to the current row are included.
```

**Flow:**

1. Sales Table
2. SELECT ProductID,SUM(Quantity) OVER(PARTITION BY ProductIDORDER BY SaleDate ROWSBETWEEN UNBOUNDEDPRECEDING AND CURRENTROW) AS RunningTotal
3. Summary of Running Totalsper Product

