---
type: "practice"
difficulty: "Deep"
---

# What is denormalization and why might you deliberately denormalize a schema?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Filtered Indexes and Plan Cache](<../../../../TOOLS/SQL_Server/T-SQL/Filtered_Indexes_and_Plan_Cache.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Denormalization in T-SQL involves structuring your database to reduce the number of joins or to store redundant data, potentially at the cost of violating normalization rules like third normal form (3NF). You might deliberately denormalize a schema when you need faster query performance and simpler join logic, even though it can lead to larger storage requirements and potential data redundancy. In this context, we'll explore what denormalization is, why it's used, and how to implement it in T-SQL.

**Code:**

```sql
-- Example of a normalized schema
CREATE TABLE Orders (
    OrderID INT PRIMARY KEY,
    CustomerID INT,
    OrderDate DATE
)

CREATE TABLE Customers (
    CustomerID INT PRIMARY KEY,
    FirstName NVARCHAR(50),
    LastName NVARCHAR(50)
)

-- Denormalized version to improve performance
CREATE TABLE SalesSummary (
    OrderID INT,
    CustomerName NVARCHAR(100),  -- Concatenated name for simplicity
    OrderDate DATE,
    TotalAmount DECIMAL(18,2)
)
```

**Example input / output:**

```

-- Insert data into the normalized schema INSERT INTO Orders (OrderID, CustomerID, OrderDate) VALUES (1, 101, '2023-01-01'), (2, 102, '2023-01-02')

INSERT INTO Customers (CustomerID, FirstName, LastName) VALUES (101, 'John', 'Doe'), (102, 'Jane', 'Smith')

CODE
Copy

```sql
-- Insert denormalized data into the SalesSummary table
INSERT INTO SalesSummary (OrderID, CustomerName, OrderDate, TotalAmount)
VALUES
(1, 'John Doe', '2023-01-01', 150.00),
(2, 'Jane Smith', '2023-01-02', 200.00)
```

**Flow:**

1. Normalized Schema
2. Orders Table
3. Customers Table
4. SalesSummary DenormalizedTable

