---
type: "practice"
difficulty: "Basic"
---

# What are OLAP and OLTP, and which schema style suits each?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ OLTP vs Star Schema Performance](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/OLTP_vs_Star_Schema_Performance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

OLAP (Online Analytical Processing) and OLTP (Online Transactional Processing) refer to two different types of database operations with distinct use cases. OLAP is typically used for business intelligence tasks, such as generating reports and analyzing historical data, whereas OLTP systems handle transactional operations like customer orders and account management in real-time.

In terms of schema design, Star Schema and Snowflake Schema are two popular approaches within Data Modeling. Star Schema consists of a central fact table surrounded by dimension tables, while Snowflake Schema normalizes the dimension tables to create a more complex structure with multiple levels.

For OLAP systems, Star Schema is often preferred because it provides a simple and efficient way to query large datasets. On the other hand, OLTP systems benefit from using normalized schemas like Snowflake due to their need for data integrity and transactional consistency.

Let's break down these concepts step-by-step:

Understanding OLAP vs. OLTP:

OLAP: This type of database is optimized for read-heavy operations that involve complex queries, such as reporting and analytics.
OLTP: This is designed for write-heavy transactions, where data must be inserted, updated, or deleted in near real-time.

Star Schema:

Fact Table: Stores the primary metrics of interest.
Dimension Tables: Contain descriptive information about the fact table records.
Simplicity and Performance: Since all relevant dimensions are denormalized into a single fact table, Star Schema is easier to query and optimizes for fast reads.

Snowflake Schema:

Normalized Dimensions: Dimension tables are further normalized, leading to a more complex schema with multiple levels.
Data Integrity and Consistency: Snowflake schema ensures better data integrity by normalizing dimensions, making it ideal for OLTP systems that require strong transactional consistency.

Choosing the Right Schema:

For OLAP Systems, Star Schema is suitable because it simplifies complex queries and reduces the number of joins required.
For OLTP Systems, Snowflake Schema is more appropriate as it maintains data integrity, supports efficient concurrent transactions, and facilitates better management of large datasets.

#### Example Scenario
Consider a retail store's sales system. The OLTP database would use a Snowflake schema to ensure transactional consistency and maintain detailed records for each sale. In contrast, the analytics team using an OLAP system might prefer a Star Schema to easily aggregate sales data across various dimensions like time, location, and product categories.

**Comparison:**

| Star Schema Example |
|---------------------|
| **Fact Table**       |
| Sales (SaleID, Quantity, Price)     |
| **Dimension Tables**      |
| Customers (CustomerID, FirstName, LastName)     |
| Products (ProductID, Name, Category) |
| Snowflake Schema Example |
|--------------------------|
| **Primary Fact Table**    |
| Sales_Fact (SaleID, CustomerID, Date)     |
| **Normalized Dimension Tables**       |
| Customers (CustomerID, FirstName, LastName)     |
| Products (ProductID, Name, Category)

**Code:**

```sql
-- Example of Star Schema in SQL

CREATE TABLE Sales (
    SaleID INT PRIMARY KEY,
    ProductID INT,
    Quantity INT,
    Price DECIMAL(10, 2),
    FOREIGN KEY (ProductID) REFERENCES Products(ProductID)
);

CREATE TABLE Customers (
    CustomerID INT PRIMARY KEY,
    FirstName VARCHAR(50),
    LastName VARCHAR(50)
);

CREATE TABLE Sales_Fact (
    SaleID INT PRIMARY KEY,
    CustomerID INT,
    Date DATE,
    FOREIGN KEY (SaleID) REFERENCES Sales(SaleID),
    FOREIGN KEY (CustomerID) REFERENCES Customers(CustomerID)
);
```

**Example input / output:**

```text
Input (Star Schema):
SELECT s.Quantity * p.Price AS TotalRevenue,
       c.FirstName || ' ' || c.LastName AS CustomerName
FROM Sales s
JOIN Products p ON s.ProductID = p.ProductID
JOIN Customers c ON s.CustomerID = c.CustomerID;

Output:
TotalRevenue | CustomerName
-------------|--------------
89.97        | John Doe
99.98        | Jane Smith

Input (Snowflake Schema):
SELECT sf.Quantity * pd.Price AS TotalRevenue,
       cu.FirstName || ' ' || cu.LastName AS CustomerName
FROM Sales_Fact sf
JOIN Products pd ON sf.ProductID = pd.ProductID
JOIN Customers cu ON sf.CustomerID = cu.CustomerID;

Output:
TotalRevenue | CustomerName
-------------|--------------
89.97        | John Doe
99.98        | Jane Smith
```

**Data:**

```json
{
  "Sales": [
    {"SaleID": 1, "ProductID": 501, "Quantity": 3, "Price": 29.99},
    {"SaleID": 2, "ProductID": 502, "Quantity": 2, "Price": 49.99}
  ],
  "Customers": [
    {"CustomerID": 1, "FirstName": "John", "LastName": "Doe"},
    {"CustomerID": 2, "FirstName": "Jane", "LastName": "Smith"}
  ],
  "Sales_Fact": [
    {"SaleID": 1, "CustomerID": 1, "Date": "2023-10-05"},
    {"SaleID": 2, "CustomerID": 2, "Date": "2023-10-06"}
  ]
}
```

**Flow:**

1. OLTP System
2. OLTP Transaction
3. Snowflake Schema NormalizedDim Tables
4. OLAP System
5. OLAP Query
6. Star Schema Fact Table +Dimensions

