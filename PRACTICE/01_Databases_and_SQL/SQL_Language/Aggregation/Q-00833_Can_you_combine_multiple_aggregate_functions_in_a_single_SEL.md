---
type: "practice"
difficulty: "Basic"
---

# Can you combine multiple aggregate functions in a single SELECT statement? Give an example.

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Aggregate Precision and Types](<../../../../01_Databases_and_SQL/SQL_Language/Aggregation/Aggregate_Precision_and_Types.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In SQL, it is indeed possible to combine multiple aggregate functions within a single SELECT statement. This allows us to calculate and retrieve various summary statistics from our data simultaneously. For instance, we can use both COUNT, which counts the number of rows, and AVG, which calculates the average value of a column.

To illustrate this concept, let's consider an example where we have a table named Orders with columns such as OrderID, ProductID, and Quantity.

#### Triaging the Question

The core of this question is to show how you can perform multiple aggregate operations in one query. The interviewer might be probing your understanding of SQL syntax, your ability to combine different functions logically, and perhaps some basic knowledge about database normalization and data retrieval.

#### First-Principles Solution

Assuming we want to find out the total number of orders (using COUNT) along with the average quantity per order (using AVG), the query would look something like this:

SQL
SELECT
    COUNT(*) AS TotalOrders,
    AVG(Quantity) AS AvgQuantityPerOrder
FROM Orders;

#### Closing Summary
and Common Mistakes/Follow-Ups

This simple example demonstrates how to use multiple aggregate functions in a single SQL statement. It’s important to ensure that the selected columns from different aggregate functions are meaningful and useful for your analysis.

#### Common mistakes
include using aggregate functions on non-numeric fields or not including all necessary columns, which can lead to errors like Invalid Column Name. Always test your query with sample data to catch such issues early.

**Code:**

```sql
SELECT
    COUNT(*) AS TotalOrders,
    AVG(Quantity) AS AvgQuantityPerOrder
FROM Orders;
```

