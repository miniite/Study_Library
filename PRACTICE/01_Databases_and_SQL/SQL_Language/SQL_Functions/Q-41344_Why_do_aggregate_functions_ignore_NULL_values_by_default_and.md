---
type: "practice"
difficulty: "Deep"
---

# Why do aggregate functions ignore NULL values by default and how can this cause misleading results?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ NULL vs Zero in Aggregates](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Functions/NULL_vs_Zero_in_Aggregates.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

Aggregate functions in SQL, such as SUM, AVG, or COUNT, typically ignore NULL values to avoid returning invalid results. However, this behavior can lead to misleading outcomes if not properly understood and managed, especially when dealing with incomplete data.

#### Triaging the Question

The core of the question revolves around understanding why SQL aggregate functions disregard NULL values and the implications of this default behavior on query results. The interviewer is probing your knowledge of how SQL handles NULL values in aggregates, as well as potential pitfalls that might arise due to such behavior.

#### Assumptions

You are familiar with basic SQL operations and understand what NULL means.
We will consider common aggregate functions like SUM, AVG, COUNT, etc.
Detailed Solution
Why Aggregate Functions Ignore NULL Values

Avoiding Invalid Results: The primary reason for ignoring NULL values in aggregates is to ensure the results are meaningful and valid. For instance, if you use SUM, adding a non-numeric value (like NULL) could result in an incorrect sum.

Simplicity of Aggregation Logic: Ignoring NULL simplifies the logic for many aggregate functions. It prevents having to deal with special cases where NULL might affect the outcome.

#### Cons
istency Across Databases: Most SQL databases implement this behavior consistently, making it easier to write portable queries.

#### Example Scenario

Let's consider a simple table Sales with columns SalesID, ProductID, and QuantitySold.

SQL
CREATE TABLE Sales (
    SalesID INT,
    ProductID INT,
    QuantitySold INT
);

INSERT INTO Sales (SalesID, ProductID, QuantitySold)
VALUES (1, 101, NULL), (2, 102, 5), (3, 103, 7);

If you run a query to sum QuantitySold:

SQL
SELECT SUM(QuantitySold) AS TotalSales FROM Sales;

The result would be NULL, as the aggregate function ignores the NULL value.

Implications and Misleading Results

Missing Data: If your data contains missing values (i.e., NULLs), ignoring them might lead to underestimating the true total or average, which can be misleading.

Partial Aggregation: In some cases, you may want to include NULL in the aggregation to get a more accurate picture of the data. For example:

SQL
SELECT COUNT(*) AS TotalRecords, SUM(QuantitySold) AS TotalSales FROM Sales;

This query returns both the total number of records and the sum of non-null sales quantities.

Handling NULLs: To avoid misleading results, you can explicitly handle NULL values using COALESCE, IFNULL, or other functions depending on your SQL dialect:

SQL
SELECT SUM(COALESCE(QuantitySold, 0)) AS TotalSales FROM Sales;

This ensures that even if a row has a NULL value for QuantitySold, it is treated as 0.

#### Common Mistakes
and Follow-Ups
Overlooking NULL Values: Many developers might overlook the impact of NULLs on aggregate functions, leading to incorrect conclusions about data totals.
Ignoring Explicit Handling: Relying too heavily on default behavior without considering edge cases can lead to subtle bugs in complex queries.

#### Closing Summary

Understanding why SQL aggregate functions ignore NULL values is crucial for writing accurate and meaningful queries. However, this behavior can sometimes produce misleading results if the data includes missing or incomplete information. It's important to be aware of these subtleties and use appropriate handling methods when necessary.

**Comparison:**

| Sales Table |
|-------------|
| SalesID     | ProductID  | QuantitySold |
| ----------- | ---------- | ------------ |
| 1           | 101        | NULL         |
| 2           | 102        | 5            |
| 3           | 103        | 7            |

**Code:**

```

CREATE TABLE Sales ( SalesID INT, ProductID INT, QuantitySold INT );

INSERT INTO Sales (SalesID, ProductID, QuantitySold) VALUES (1, 101, NULL), (2, 102, 5), (3, 103, 7);

CODE
Copy

```sql
SELECT SUM(QuantitySold) AS TotalSales FROM Sales;
```

**Example input / output:**

```text
Input:
SELECT SUM(QuantitySold) AS TotalSales FROM Sales;

Output:
TotalSales
NULL

Explanation:
The aggregate function SUM ignores the NULL value, resulting in a total of NULL.
```

**Data:**

```plaintext
Sales table:
+---------+----------+-------------+
| SalesID | ProductID | QuantitySold |
+---------+----------+-------------+
|       1 |      101 | NULL        |
|       2 |      102 |          5  |
|       3 |      103 |          7  |
+---------+----------+-------------+
```

**Flow:**

1. Yes
2. No
3. Sales Table
4. Contains NULL?
5. Ignore NULL in Aggregate
6. Include NULL in Aggregate
7. (SUM, AVG) Result
8. (COUNT) Result

