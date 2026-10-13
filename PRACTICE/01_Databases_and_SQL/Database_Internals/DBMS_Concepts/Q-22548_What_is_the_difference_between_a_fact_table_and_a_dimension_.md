---
type: "practice"
difficulty: "Basic"
---

# What is the difference between a fact table and a dimension table in a data warehouse?

**Difficulty:** Basic · **Levels:** Fresher · [↑ Late Arriving Dimensions](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Late_Arriving_Dimensions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a data warehouse, understanding the distinction between fact tables and dimension tables is crucial for designing an effective schema. A fact table records quantitative measurements or facts about business activities, whereas dimension tables contain qualitative attributes that describe those facts in more detail.

To understand this concept from first principles, we need to break down what these terms mean:

Fact Table: This table captures the measurable events or transactions within a business process.
Dimension Table: These tables provide additional context and details about the fact records, such as time periods, locations, product information, etc.

Let's explore each of these concepts in more detail:

A fact table is often at the center of an OLAP (Online Analytical Processing) cube or a star schema.
Dimension tables are typically normalized to avoid redundancy and improve data integrity.
Fact tables can have multiple dimensions that link them to related dimension tables.

Now, let's illustrate this with an example. Imagine you're building a data warehouse for an e-commerce company:

Fact Table: Sales_Fact
Columns: SalesID, ProductID, CustomerID, StoreID, OrderDate, UnitsSold, TotalRevenue
Dimension Tables:
Products: Contains product-related attributes like ProductID, ProductName, CategoryID
Customers: Contains customer-related attributes like CustomerID, Name, City
Stores: Contains store-related attributes like StoreID, StoreName, Location
Time: Contains time-related attributes like OrderDate, Year, Month

Each fact table in a star schema is surrounded by several dimension tables. For instance, the Sales_Fact table has multiple dimensions (Products, Customers, Stores, Time) that provide context to the sales records.

In conclusion, while fact tables focus on storing measurable data, dimension tables capture non-measurable but relevant details. This separation helps in organizing and querying large volumes of data efficiently.

#### Common mistakes
include mixing up the roles of fact and dimension tables or failing to normalize dimensions properly.

**Comparison:**

| Fact Table | Dimensions |
|------------|------------|
| Sales_Fact | Products, Customers, Stores, Time |
| SalesID | ProductID | CustomerID | StoreID | OrderDate   | UnitsSold | TotalRevenue |
|---------|-----------|------------|---------|-------------|-----------|--------------|
| 100     | 5         | 200        | 1       | 2023-06-01  | 10        | 98.75        |
| ProductID | ProductName | CategoryID |
|-----------|-------------|------------|
| 5         | Laptop      | 1          |
| CustomerID | Name    | City   |
|------------|---------|--------|
| 200        | John Doe | New York|
| StoreID | StoreName | Location    |
|---------|-----------|-------------|
| 1       | Manhattan | Manhattan, NY|
| OrderDate   | Year | Month |
|-------------|------|-------|
| 2023-06-01  | 2023 | 6     |

**Code:**

```sql
-- Example schema for a star schema
CREATE TABLE Sales_Fact (
    SalesID INT,
    ProductID INT,
    CustomerID INT,
    StoreID INT,
    OrderDate DATE,
    UnitsSold INT,
    TotalRevenue DECIMAL(10,2)
);

CREATE TABLE Products (
    ProductID INT PRIMARY KEY,
    ProductName VARCHAR(50),
    CategoryID INT
);

CREATE TABLE Customers (
    CustomerID INT PRIMARY KEY,
    Name VARCHAR(100),
    City VARCHAR(100)
);

CREATE TABLE Stores (
    StoreID INT PRIMARY KEY,
    StoreName VARCHAR(100),
    Location VARCHAR(100)
);

CREATE TABLE Time (
    OrderDate DATE PRIMARY KEY,
    Year INT,
    Month INT
);
```

**Flow:**

1. Dimensions
2. FactTable
3. ProductDim
4. CustomerDim
5. StoreDim
6. TimeDim

