---
type: "practice"
difficulty: "Intermediate"
---

# Why does a normalized OLTP schema make analytical queries slower than a star schema?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ OLTP vs Star Schema Performance](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/OLTP_vs_Star_Schema_Performance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this answer, we'll explore why a normalized Online Transaction Processing (OLTP) schema often results in slower analytical queries compared to a star schema. We'll start by understanding what these schemas are and then delve into the reasons behind their performance differences.

#### Triage

The question is probing our understanding of data normalization, OLTP schemas, star schemas, and how different database designs affect query performance. The assumptions here include familiarity with basic database concepts such as ACID properties, indexing, and join operations. We need to reason through why a denormalized (star) schema can provide faster query times despite the initial increase in storage requirements.

#### First Principles Solution
Normalization vs Denormalization: Normalization aims to reduce data redundancy and improve data integrity by splitting tables into smaller pieces. In contrast, denormalization intentionally increases redundancy for performance benefits.
Join Operations:
OLTP (Normalized Schema): Typically involves multiple joins between related tables to retrieve the required information. This can be time-consuming as each join operation adds latency.
Star Schema: Uses a central fact table surrounded by dimension tables, minimizing the number of joins needed for queries.
Indexing and Query Optimization:
OLTP databases often have extensive indexing but are still prone to slow query execution due to the complexity introduced by multiple joins.
Star schemas can be optimized with fewer, more strategic indexes.
Detailed Explanation

Let's consider a simplified example where we need to retrieve sales data for a specific product category and date range. We'll compare this process in an OLTP schema versus a star schema.

OLTP Schema Example (Normalized)
SQL
CREATE TABLE Orders (
    OrderID INT PRIMARY KEY,
    ProductID INT NOT NULL,
    Quantity INT NOT NULL,
    OrderDate DATETIME NOT NULL
);

CREATE TABLE Products (
    ProductID INT PRIMARY KEY,
    CategoryID INT NOT NULL,
    Name VARCHAR(255) NOT NULL,
    Price DECIMAL(10, 2)
);

CREATE TABLE Categories (
    CategoryID INT PRIMARY KEY,
    Name VARCHAR(255) NOT NULL
);
Star Schema Example (Denormalized)
SQL
CREATE TABLE Sales (
    SaleID INT PRIMARY KEY,
    ProductID INT NOT NULL,
    Quantity INT NOT NULL,
    SaleDate DATETIME NOT NULL,
    CategoryName VARCHAR(255) -- denormalized field
);

CREATE TABLE Categories (
    CategoryID INT PRIMARY KEY,
    Name VARCHAR(255) NOT NULL
);
Query Comparison
OLTP Schema Query
SQL
SELECT p.Name, SUM(o.Quantity)
FROM Orders o
JOIN Products p ON o.ProductID = p.ProductID
WHERE o.OrderDate BETWEEN '2023-01-01' AND '2023-12-31'
GROUP BY p.Name;
Star Schema Query
SQL
SELECT CategoryName, SUM(Quantity)
FROM Sales s
JOIN Categories c ON s.CategoryID = c.CategoryID
WHERE SaleDate BETWEEN '2023-01-01' AND '2023-12-31'
GROUP BY CategoryName;
Why Star Schema is Faster
Reduced Joins: The star schema query involves fewer joins, reducing the complexity and latency.
Denormalized Fields: Pre-aggregated fields like CategoryName in the sales table can be used directly without additional lookups.
Indexing Efficiency: With proper indexing on common join keys (e.g., ProductID, SaleDate), OLTP queries may still perform well, but star schemas often have more straightforward index patterns that can lead to faster execution.

#### Common Mistakes
and Follow-Ups
Over-normalization: Often, data engineers normalize too much, leading to complex queries.
Indexing Strategy: Incorrect or insufficient indexing in OLTP databases can negate performance gains from normalization.
Query Optimization Techniques: Advanced query optimization techniques might not fully mitigate the performance hit of OLTP joins.

In conclusion, while normalized schemas are crucial for maintaining data integrity and reducing redundancy, they can introduce overhead that impacts analytical query performance. Star schemas, despite their increased storage costs, often provide a better balance between query speed and storage efficiency in many real-world scenarios.

**Comparison:**

```
OLTP Schema
Join Orders and Products
Select Product Names andQuantities
Retrieve Sales Data for aSpecific Date Range
GROUP BY Product Name
Star Schema
Directly Select from Sales Table
Select CategoryNames andTotal Quantities
```

**Code:**

```

-- OLTP schema example CREATE TABLE Orders ( OrderID INT PRIMARY KEY, ProductID INT NOT NULL, Quantity INT NOT NULL, OrderDate DATETIME NOT NULL );

CREATE TABLE Products ( ProductID INT PRIMARY KEY, CategoryID INT NOT NULL, Name VARCHAR(255) NOT NULL, Price DECIMAL(10, 2) );

CREATE TABLE Categories ( CategoryID INT PRIMARY KEY, Name VARCHAR(255) NOT NULL );

CODE
Copy

```sql
-- Star schema example
CREATE TABLE Sales (
    SaleID INT PRIMARY KEY,
    ProductID INT NOT NULL,
    Quantity INT NOT NULL,
    SaleDate DATETIME NOT NULL,
    CategoryName VARCHAR(255) -- denormalized field
);

CREATE TABLE Categories (
    CategoryID INT PRIMARY KEY,
    Name VARCHAR(255) NOT NULL
);
```

**Example input / output:**

```text
-- OLTP schema query
SELECT p.Name, SUM(o.Quantity) as TotalQuantity
FROM Orders o
JOIN Products p ON o.ProductID = p.ProductID
WHERE o.OrderDate BETWEEN '2023-01-01' AND '2023-12-31'
GROUP BY p.Name;

-- Star schema query
SELECT CategoryName, SUM(Quantity) as TotalQuantity
FROM Sales s
JOIN Categories c ON s.CategoryID = c.CategoryID
WHERE SaleDate BETWEEN '2023-01-01' AND '2023-12-31'
GROUP BY CategoryName;
```

**Data:**

```

-- Example data for OLTP schema INSERT INTO Orders (OrderID, ProductID, Quantity, OrderDate) VALUES (1, 101, 5, '2023-01-01'), (2, 102, 4, '2023-02-01');

INSERT INTO Products (ProductID, CategoryID, Name, Price) VALUES (101, 1, 'Laptop', 999.99), (102, 1, 'Monitor', 399.99);

INSERT INTO Categories (CategoryID, Name) VALUES (1, 'Electronics');

CODE
Copy

```sql
-- Example data for Star schema
INSERT INTO Sales (SaleID, ProductID, Quantity, SaleDate, CategoryName)
VALUES (101, 101, 5, '2023-01-01', 'Electronics'),
       (102, 102, 4, '2023-02-01', 'Electronics');

INSERT INTO Categories (CategoryID, Name)
VALUES (1, 'Electronics');
```

**Flow:**

1. OLTP Query
2. Join Orders and Products
3. Select Product Names andQuantities
4. Retrieve Sales Data for aSpecific Date Range
5. GROUP BY Product Name
6. Star Query
7. Directly Select from Sales Table
8. Select CategoryNames andTotal Quantities

