---
type: "practice"
difficulty: "Intermediate"
---

# Why are OLTP databases usually normalized but OLAP data warehouses often use star schemas?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ OLTP vs Star Schema Performance](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/OLTP_vs_Star_Schema_Performance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand why OLTP databases are typically normalized while OLAP data warehouses often employ star and snowflake schemas, we need to delve into the fundamental differences in their design goals and operational needs.

OLTP (Online Transaction Processing) systems aim for fast transactional operations such as insertions, updates, and queries that support day-to-day business activities. They require minimal redundancy to avoid anomalies like insertion, update, and deletion anomalies. Normalization ensures data integrity and consistency by minimizing data duplication.

On the other hand, OLAP (Online Analytical Processing) systems are designed for complex analytical operations such as reporting, querying large volumes of historical data, and providing comprehensive business intelligence insights. Star and snowflake schemas simplify query performance through a star schema's central fact table surrounded by dimension tables or through a more denormalized structure in a snowflake schema.

Let’s break this down step-by-step:

Normalization in OLTP:

Normalization reduces data redundancy to minimize insertion, update, and deletion anomalies.
It ensures that each piece of information is stored only once, making the database smaller and more efficient for transactional workloads.

Star Schema in OLAP:

A star schema consists of a central fact table and several dimension tables.
The central fact table stores numerical data related to transactions or events (e.g., sales figures, inventory levels).
Dimension tables store descriptive attributes that are related to the fact table (e.g., customer information, time periods).

Snowflake Schema:

A snowflake schema is an extension of a star schema where dimension tables are further normalized into multiple tables.
This creates a more hierarchical structure with more granularity but can lead to more complex queries.

#### Performance Considerations

OLTP systems prioritize write operations, so normalization helps maintain data integrity during these frequent transactions.
OLAP systems prioritize read operations for analysis and reporting, so star/snowflake schemas allow for simpler and faster query execution by reducing join complexity.

Storage and Maintenance:

Normalized OLTP databases are more efficient in terms of storage as they avoid redundant data.
Star and snowflake schemas can be larger due to the additional dimension tables but offer better query performance through reduced joins and aggregations.

In summary, normalization is ideal for OLTP systems because it maintains data consistency during frequent transactions. In contrast, star and snowflake schemas are preferred in OLAP environments where complex queries and large datasets are common, offering better performance through optimized schema design.

**Comparison:**

```
StarSchema
Fact Table
Dimension 1
Dimension 2
```

**Code:**

```sql
-- Example of a simple Star Schema
CREATE TABLE Sales (
    SaleID INT PRIMARY KEY,
    CustomerID INT,
    ProductID INT,
    Quantity INT,
    Price DECIMAL(10, 2),
    FOREIGN KEY (CustomerID) REFERENCES Customers(CustomerID),
    FOREIGN KEY (ProductID) REFERENCES Products(ProductID)
);

CREATE TABLE Customers (
    CustomerID INT PRIMARY KEY,
    FirstName VARCHAR(50),
    LastName VARCHAR(50),
    Address VARCHAR(100)
);

CREATE TABLE Products (
    ProductID INT PRIMARY KEY,
    Name VARCHAR(100),
    Category VARCHAR(50)
);
```

**Example input / output:**

```text
Input:
 - Fact table (Sales)
 - Dimension tables (Customers and Products)

 Output:
 - Query to retrieve total sales by customer:
 SELECT c.FirstName, c.LastName, SUM(s.Quantity * s.Price) AS TotalSales
 FROM Sales s
 JOIN Customers c ON s.CustomerID = c.CustomerID
 GROUP BY c.CustomerID;

 - Query to fetch all product details:
 SELECT p.Name, p.Category, COUNT(*) AS SaleCount
 FROM Sales s
 JOIN Products p ON s.ProductID = p.ProductID
 GROUP BY p.ProductID;
```

**Data:**

```json
{
   "Sales": [
     {"SaleID": 1, "CustomerID": 101, "ProductID": 201, "Quantity": 3, "Price": 9.99},
     {"SaleID": 2, "CustomerID": 102, "ProductID": 202, "Quantity": 2, "Price": 4.99}
   ],
   "Customers": [
     {"CustomerID": 101, "FirstName": "John", "LastName": "Doe", "Address": "123 Main St"},
     {"CustomerID": 102, "FirstName": "Jane", "LastName": "Smith", "Address": "456 Elm St"}
   ],
   "Products": [
     {"ProductID": 201, "Name": "Laptop", "Category": "Electronics"},
     {"ProductID": 202, "Name": "Keyboard", "Category": "Accessories"}
   ]
 }
```

**Flow:**

1. DimensionTables
2. FactTable
3. Sales
4. Customers
5. Products

