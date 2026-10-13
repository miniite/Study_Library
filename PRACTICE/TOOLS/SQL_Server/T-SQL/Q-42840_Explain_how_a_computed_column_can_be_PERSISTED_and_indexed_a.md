---
type: "practice"
difficulty: "Deep"
---

# Explain how a computed column can be PERSISTED and indexed, and give a case where it improved query performance.

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Temp Tables vs Table Variables](<../../../../TOOLS/SQL_Server/T-SQL/Temp_Tables_vs_Table_Variables.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Computed columns in T-SQL are virtual columns that are derived from one or more columns in the same table. When a computed column is persisted, its values are physically stored in the table, alongside other data. This can significantly improve query performance if the computed value is frequently used and calculated on-the-fly would be expensive.

To answer this question, let's first define what it means to persist a computed column, then discuss how indexing such columns can further enhance performance, and finally provide an example scenario where persistence and indexing led to significant improvements.

#### Triaging the Question

The core of the question is about understanding when and why one would want to persist a computed column in T-SQL, and how indexing can be leveraged to optimize queries that use this column. The interviewer will likely probe your knowledge on:

What persists means in terms of storage.
How SQL Server calculates and stores the values for persisted versus non-persisted computed columns.
The impact of indexing a persisted computed column.
Scenarios where such optimizations are beneficial.

#### First Principles Solution

Understanding Computed Columns:

A computed column is derived from other columns in the same table using an expression.
For example, Sales = Price * Quantity could be a computed column for tracking total sales per row.

Persisted vs Non-Persisted Computed Columns:

By default, computed columns are non-persisted, meaning SQL Server recalculates the value each time it is queried.
Persisting a computed column means storing its result in the table, which can be more efficient for frequently accessed values.

Indexing Persisted Columns:

Indexing a persisted computed column allows faster retrieval of data based on this column's values.
This can significantly speed up queries that filter or sort by the computed value.

Case Study: Sales Data Table

#### Cons
ider a Sales table with columns Price, Quantity, and Sales.
The Sales column is defined as a persisted computed column using the expression Price * Quantity.

#### Example Scenario

Let's say we have a Sales table used in an e-commerce application where sales are frequently reported by product. We define the Sales column as:

SQL
CREATE TABLE Sales (
    ProductID INT,
    Price DECIMAL(10,2),
    Quantity INT,
    Sales AS (Price * Quantity) PERSISTED
);
Scenario: Reporting Daily Sales
The application frequently needs to generate reports showing total sales for each product.
Without persistence, SQL Server would need to compute Sales on the fly every time a report is generated.

By persisting the computed column and creating an appropriate index:

SQL
CREATE INDEX idx_Sales ON Sales (Sales);
Performance Improvement

Query Optimization:

Queries like SELECT * FROM Sales WHERE Sales > 500 now benefit from the persisted value.
The database can directly use the pre-calculated values, reducing the load on compute resources.

Real-World Impact:

In a large e-commerce platform with millions of transactions daily, persisting and indexing Sales could lead to faster report generation times, improving user experience and allowing more efficient resource management.

#### Closing Summary

Persisting computed columns can offer significant performance benefits by reducing the overhead of recalculating values frequently. Properly indexed persisted computed columns can greatly enhance query performance, especially in scenarios where derived data is often used for filtering or sorting. However, it's important to consider the storage implications and potential updates required, as persisting a column adds additional space and complexity.

#### Common Mistakes
and Follow-Ups

Mistake: Persisting non-critical computed columns that are rarely used.

This can lead to unnecessary storage overhead with little benefit.

#### Follow-Up
How would you handle cases where the base data changes frequently?

Ensure the persisted column is updated in a timely manner, possibly through triggers or other mechanisms.

**Comparison:**

```
ProductID	Price	Quantity	Sales
1	50.50	20	1010.00
```

**Code:**

```sql
-- Creating the Sales table with a persisted computed column
CREATE TABLE Sales (
    ProductID INT,
    Price DECIMAL(10,2),
    Quantity INT,
    Sales AS (Price * Quantity) PERSISTED
);

-- Indexing the persisted computed column for faster query performance
CREATE INDEX idx_Sales ON Sales (Sales);
```

**Example input / output:**

```text
- **Input:** `SELECT * FROM Sales WHERE Sales > 500`
- **Output:** Rows with `Sales` values greater than 500, leveraging the persisted and indexed column for faster results.
```

**Data:**

```json
{
  "ProductID": 1,
  "Price": 50.50,
  "Quantity": 20,
  "Sales": 1010.00
}
```

**Flow:**

1. Yes
2. No
3. Sales Table Creation
4. Persist Computed Column?
5. Persist and Index Sales Column
6. Use Non-Persisted ComputedColumn
7. Generate Fast Queries UsingIndexed Values
8. Recalculate on the Fly, SlowerQueries

