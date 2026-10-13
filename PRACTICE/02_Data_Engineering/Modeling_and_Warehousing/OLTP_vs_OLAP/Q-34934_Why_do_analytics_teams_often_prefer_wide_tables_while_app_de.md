---
type: "practice"
difficulty: "Intermediate"
---

# Why do analytics teams often prefer 'wide' tables while app developers prefer 'narrow' related tables?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Physical Design for OLTP vs OLAP](<../../../../02_Data_Engineering/Modeling_and_Warehousing/OLTP_vs_OLAP/Physical_Design_for_OLTP_vs_OLAP.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In database design, 'wide' and 'narrow' table structures serve different purposes and are preferred in various contexts. Analytical teams often favor wide tables because they provide a comprehensive view of data, which is ideal for complex queries and reports. Conversely, application developers prefer narrow, related tables to optimize read performance and simplify data manipulation.

To understand why these preferences exist, we need to break down the core concepts of OLTP (Online Transaction Processing) and OLAP (Online Analytical Processing), and how different applications leverage each type of database design.

**Code:**

```

-- Example schema for narrow tables in an OLTP system CREATE TABLE Orders ( OrderID INT PRIMARY KEY, CustomerID INT NOT NULL, ProductID INT NOT NULL, Quantity INT NOT NULL, OrderDate DATE NOT NULL );

CREATE TABLE Customers ( CustomerID INT PRIMARY KEY, FirstName VARCHAR(50) NOT NULL, LastName VARCHAR(50) NOT NULL, Email VARCHAR(100) );

CODE
Copy

```sql
-- Example schema for wide tables in an OLAP system
CREATE TABLE SalesData (
    Year INT NOT NULL,
    Month INT NOT NULL,
    Day INT NOT NULL,
    CustomerID INT NOT NULL,
    ProductID INT NOT NULL,
    Quantity INT NOT NULL,
    Price DECIMAL(10, 2) NOT NULL,
    Revenue DECIMAL(15, 4)
);
```

**Example input / output:**

```

Here’s a worked input-output scenario to illustrate the difference:

OLTP Application Example

Input: User places an order with multiple items.
SQL
Copy
INSERT INTO Orders (OrderID, CustomerID, ProductID, Quantity, OrderDate) VALUES (1001, 502, 603, 2, '2023-09-01');
Output: Data stored in the Orders table.
SQL
Copy
SELECT * FROM Orders WHERE OrderID = 1001;

OLAP Application Example

Input: Generate a report on sales for Q3 of 2023.
SQL
Copy
SELECT Year, Month, Day, SUM(Quantity) AS TotalUnitsSold, SUM(Revenue) AS TotalSales
FROM SalesData WHERE Year = 2023 AND Month BETWEEN 7 AND 9 GROUP BY Year, Month, Day;
Output: A summarized report with total units sold and revenue for each day in Q3.
```

