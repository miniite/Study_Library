---
type: "practice"
difficulty: "Deep"
---

# Analysts join two fact tables directly and get a fan-out that multiplies revenue; how do you explain the fan trap and redesign so drill-across goes through conformed dimensions instead?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Aggregate Fact Tables](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Aggregate_Fact_Tables.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

The issue described is known as a "fan trap" in data modeling, where direct joins between two fact tables lead to an exponential increase in the number of records, making queries slow and resource-intensive. To resolve this, we should redesign our schema using conformed dimensions to enable efficient drill-across operations.

#### Triaging the Question

The core of the question is about optimizing a data model for performance when multiple fact tables need to be joined together frequently. The key points being probed are:

Understanding of Fan Trap: Identifying and explaining why direct joins between two fact tables lead to significant performance issues.
Concept of Conformed Dimensions: Understanding how conformed dimensions can help in reducing the fan-out problem and enabling efficient query execution.

#### First-Principles Solution
Step 1: Identify the Issue (Fan Trap)

When analysts join two fact tables directly, each record in one table is joined with every record in the other table. If there are (n) records in each fact table, this results in (n \times n = n^2) possible combinations. This quadratic growth can severely degrade query performance.

Step 2: Introduce Conformed Dimensions

Conformed dimensions are shared dimension tables that multiple fact tables reference. By doing so, we can avoid the fan-out problem and allow queries to leverage indexes on the conformed dimensions for faster joins.

#### Example Scenario

Suppose you have two fact tables: Sales and Returns. Both of these tables have a common attribute called CustomerID.

Original Fact Table Design:
SQL
SELECT s.CustomerID, s.SaleAmount, r.ReturnAmount
FROM Sales AS s
JOIN Returns AS r ON s.CustomerID = r.CustomerID;

This join results in a fan-out problem because each sales record could be matched with every return record. The resulting dataset can grow exponentially.

Redesigned Schema Using Conformed Dimensions:

Create a conformed dimension table for Customer that both Sales and Returns reference.
SQL
CREATE TABLE Customer (
  CustomerID INT PRIMARY KEY,
  FirstName VARCHAR(50),
  LastName VARCHAR(50)
);
Modify the fact tables to include foreign keys referencing the conformed dimension.
SQL
ALTER TABLE Sales ADD COLUMN CustomerID INT;
ALTER TABLE Returns ADD COLUMN CustomerID INT;

-- Populate the foreign key relationships with data from the original Customer table.

Optimized Query:

SQL
SELECT s.CustomerID, s.SaleAmount, r.ReturnAmount
FROM (SELECT * FROM Sales JOIN Customer ON Sales.CustomerID = Customer.CustomerID) AS s
JOIN (SELECT * FROM Returns JOIN Customer ON Returns.CustomerID = Customer.CustomerID) AS r ON s.CustomerID = r.CustomerID;

By using the conformed dimension Customer, we ensure that the join is performed on a shared, indexed table. This reduces the number of records to be joined and significantly improves query performance.

Step 3: Evaluate Performance
Join Optimization: Use indexes on the conformed dimensions to speed up joins.
Query Plan Analysis: Monitor query execution plans and adjust as necessary.

#### Closing Summary

Redesigning the schema to use conformed dimensions effectively addresses the fan trap issue. By leveraging shared dimension tables, we can maintain data consistency while optimizing join performance. This approach is particularly beneficial in large-scale data warehousing environments where frequent cross-table queries are common.

#### Common Mistakes
and Follow-Ups
Missing Indexes: Ensure that relevant indexes on conformed dimensions are created to support efficient joins.
Data Consistency: Conformed dimensions must be kept up-to-date with the fact tables to avoid anomalies in query results.
Performance Testing: Regularly test and refine the schema based on actual workload and performance metrics.

By following these steps, you can ensure that your data model is optimized for both data integrity and query performance.

**Comparison:**

| Fact Table | Key Columns |
|------------|-------------|
| Sales      | SaleID, CustomerID, SaleAmount |
| Returns    | ReturnID, CustomerID, ReturnAmount |
| Conformed Dimension | Key Columns |
|---------------------|-------------|
| Customer            | CustomerID, FirstName, LastName |

**Code:**

```

-- Original Fact Table Design

CREATE TABLE Sales ( SaleID INT PRIMARY KEY, CustomerID INT, SaleAmount DECIMAL(10,2) );

CREATE TABLE Returns ( ReturnID INT PRIMARY KEY, CustomerID INT, ReturnAmount DECIMAL(10,2) );

CODE
Copy

```sql
-- Conformed Dimension Design

CREATE TABLE Customer (
  CustomerID INT PRIMARY KEY,
  FirstName VARCHAR(50),
  LastName VARCHAR(50)
);

ALTER TABLE Sales ADD COLUMN CustomerID INT;
ALTER TABLE Returns ADD COLUMN CustomerID INT;

-- Populate the foreign key relationships with data from the original Customer table.
SQL
Copy
-- Optimized Query

SELECT s.CustomerID, s.SaleAmount, r.ReturnAmount
FROM (SELECT * FROM Sales JOIN Customer ON Sales.CustomerID = Customer.CustomerID) AS s
JOIN (SELECT * FROM Returns JOIN Customer ON Returns.CustomerID = Customer.CustomerID) AS r ON s.CustomerID = r.CustomerID;
```

**Example input / output:**

```text
Original Query:
SELECT s.CustomerID, s.SaleAmount, r.ReturnAmount
FROM Sales AS s
JOIN Returns AS r ON s.CustomerID = r.CustomerID;

Optimized Query with Conformed Dimensions:
SELECT s.CustomerID, s.SaleAmount, r.ReturnAmount
FROM (SELECT * FROM Sales JOIN Customer ON Sales.CustomerID = Customer.CustomerID) AS s
JOIN (SELECT * FROM Returns JOIN Customer ON Returns.CustomerID = Customer.CustomerID) AS r ON s.CustomerID = r.CustomerID;
```

**Data:**

```json
{
  "Sales": {
    "SaleID": [1, 2, 3],
    "CustomerID": [101, 101, 102],
    "SaleAmount": [100.00, 200.50, 150.75]
  },
  "Returns": {
    "ReturnID": [4, 5],
    "CustomerID": [101, 102],
    "ReturnAmount": [-30.00, -50.50]
  },
  "Customer": {
    "CustomerID": [101, 102],
    "FirstName": ["Alice", "Bob"],
    "LastName": ["Smith", "Johnson"]
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. Fact Table - Sales
5. Does the fact table need tojoin with another?
6. Create Conformed Dimension
7. Join Fact Tables via ConformedDimension
8. Fact Table - Returns
9. Does the fact table need tojoin with another?
10. Create Conformed Dimension

