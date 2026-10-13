---
type: "practice"
difficulty: "Basic"
---

# What is a star schema and why is it commonly used in data warehouses?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Distribution Keys](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Data_Warehousing/Distribution_Keys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary:

A star schema is a database schema that organizes data into a central fact table surrounded by dimension tables, forming a "star" pattern. This structure simplifies querying and analysis, making it a popular choice in data warehouses.

#### Triaging the Question

The question asks about the definition of a star schema and its common usage in data warehouses. The interviewer is likely probing your understanding of database design principles and their application in data warehousing environments.

#### First-Principles Solution
What Is a Star Schema?

A star schema consists of three main components:

Fact Table: This table holds the quantitative business facts, such as sales amounts or product quantities.
Dimension Tables: These hold descriptive attributes about the fact data. For example, if the fact table includes sales information, dimension tables might include customer details, time periods, and product categories.

The schema is called a "star" because when you draw it out with the central fact table at one point, and the dimension tables radiating outwards like points of a star, it resembles a star pattern. Here's an example:

SalesAmount
CustomerID
OrderDate
Fact Table
Dimension 1
Dimension 2
Dimension 3
Why Is It Commonly Used?

Star schemas are preferred in data warehouses for several reasons:

Simplicity: The structure is straightforward and easy to understand, making it simpler to query and maintain.
Performance: Queries can be optimized by indexing the dimension tables, which significantly speeds up joins with the fact table.
Modularity: Fact tables can be reused across multiple queries without duplicating data, reducing storage needs.
Common Assumptions:
The data warehouse will store historical data for analysis purposes.
There is a need to perform complex analytical queries involving time series and other dimensions.
Performance optimization is crucial due to the volume of data being processed.

#### Closing Summary
+ Common Mistakes/Follow-Ups:

In summary, star schemas are highly effective in data warehouses because they offer simplicity, performance, and modularity. However, it's important to note that while they simplify queries, they can become less efficient if not properly indexed or if the fact table becomes too large.

#### Common mistakes
include:

Overusing star schemas for transactional databases where normalization is crucial.
Failing to index dimension tables adequately, leading to poor query performance.
Not recognizing when a snowflake schema might be more appropriate for complex hierarchies and relationships.

#### Follow-ups
could involve discussing how to handle slowly changing dimensions or scenarios where the star schema needs to evolve into a snowflake schema.

**Comparison:**

```
QuantitySold
ProductID
FactTable (SalesFact)
DimTable 1 (CustomersDim)
DimTable 2 (ProductsDim)
```

**Code:**

```sql
-- Example of a Star Schema in SQL

CREATE TABLE SalesFact (
    SaleID INT PRIMARY KEY,
    CustomerID INT NOT NULL,
    ProductID INT NOT NULL,
    OrderDate DATE NOT NULL,
    QuantitySold INT NOT NULL,
    Price DECIMAL(10, 2) NOT NULL
);

CREATE TABLE CustomersDim (
    CustomerID INT PRIMARY KEY,
    FirstName VARCHAR(50),
    LastName VARCHAR(50)
);

CREATE TABLE ProductsDim (
    ProductID INT PRIMARY KEY,
    Name VARCHAR(100),
    Category VARCHAR(50)
);

-- Example Query
SELECT
    SUM(SF.QuantitySold) AS TotalSales,
    PD.Category,
    CD.FirstName, CD.LastName
FROM SalesFact SF
JOIN CustomersDim CD ON SF.CustomerID = CD.CustomerID
JOIN ProductsDim PD ON SF.ProductID = PD.ProductID
GROUP BY PD.Category, CD.FirstName, CD.LastName;
```

