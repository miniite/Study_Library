---
type: "practice"
difficulty: "Intermediate"
---

# How do you convert an existing normalized OLTP schema into a star schema; what is your step-by-step approach for identifying facts and dimensions?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ OLTP vs Star Schema Performance](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/OLTP_vs_Star_Schema_Performance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When converting an existing normalized Online Transaction Processing (OLTP) database into a star schema, the goal is to denormalize data by grouping related tables around central fact tables and adding dimension tables as necessary. This transformation is essential for improving query performance in business intelligence systems that rely on complex analytics.

#### Triage

The question asks you to outline your step-by-step approach for identifying facts and dimensions when converting an OLTP schema into a star schema. The interviewer is probing your understanding of data modeling principles, ability to identify key relationships, and familiarity with the normalization and denormalization process.

Step-by-Step Approach
Understand the Current Schema: Begin by thoroughly understanding the existing normalized OLTP schema. Identify all the tables, their primary keys (PKs), foreign keys (FKs), and any relationships between them.
Identify Facts: A fact is a data point that measures something about the business domain, such as sales amounts or quantities sold. Typically, these are numerical values that you want to summarize over time.
Identify Dimensions: Dimensions provide context for facts. They include non-numerical attributes like customer demographics, date and time, location, product categories, etc.
Design the Star Schema:
Create a Fact table containing all relevant fact measures.
Identify candidate dimension tables from the OLTP schema that can support your star schema.
Normalize Dimensions: If necessary, further normalize dimensions to ensure they are clean and consistent. This might involve creating new lookup tables or breaking down large dimensions into smaller parts.
Denormalize Facts: Move detailed attributes (from OLTP tables) into the fact table if those attributes are frequently queried alongside the facts.

#### Example
Walkthrough

Let’s consider an example where you have a Sales table and several other related tables in your OLTP schema:

Sales Table:

PK: SaleID
FKs: CustomerID, ProductID, LocationID, DateID

Customer Table (OLTP):

PK: CustomerID
Attributes: Name, Age, Gender, etc.

Product Table (OLTP):

PK: ProductID
Attributes: ProductName, Category, Price, etc.

Location Table (OLTP):

PK: LocationID
Attributes: City, State, Country
Fact Identification: The Sales table can be the fact table since it contains numerical data about sales.
Dimension Identification: The Customer, Product, and Location tables are dimensions that provide context to the sales facts.
Star Schema Design

Fact Table: Sales

FKs from Dimensions (e.g., CustomerID, ProductID, DateID, LocationID)
Measures (e.g., TotalSalesAmount, QuantitySold)

Dimension Tables:

Customer
Product
Date
Location

#### Common Mistakes
and Follow-Ups
Mistake 1: Ignoring the importance of consistent data types between fact and dimension tables.
Mistake 2: Failing to normalize dimensions properly, leading to redundant storage.

#### Follow-Up
Discuss potential trade-offs in choosing between a star schema and a snowflake schema, depending on the complexity and size of your data.

By carefully identifying facts and dimensions and following these steps, you can effectively convert an OLTP schema into a more optimized star schema for analytics purposes.

**Comparison:**

| Fact Table (Sales)          | Dimension Tables |
|-----------------------------|------------------|
| SaleID, CustomerID, ProductID, DateID, LocationID, TotalSalesAmount, QuantitySold | Customer, Product, Date, Location |

**Code:**

```sql
-- Example Fact Table Creation in Star Schema

CREATE TABLE Sales (
    SaleID INT PRIMARY KEY,
    CustomerID INT NOT NULL,
    ProductID INT NOT NULL,
    DateID INT NOT NULL,
    LocationID INT NOT NULL,
    TotalSalesAmount DECIMAL(10,2),
    QuantitySold INT
);

-- Example Dimension Table Creation for Customer

CREATE TABLE Customer (
    CustomerID INT PRIMARY KEY,
    Name VARCHAR(50),
    Age INT,
    Gender CHAR(1)
);
```

**Example input / output:**

```text
-- Input Data

INSERT INTO Sales (SaleID, CustomerID, ProductID, DateID, LocationID, TotalSalesAmount, QuantitySold)
VALUES
(1, 101, 201, 20230101, 501, 50.00, 2),
(2, 102, 202, 20230102, 502, 75.50, 3),
(3, 103, 203, 20230103, 503, 98.25, 4);

INSERT INTO Customer (CustomerID, Name, Age, Gender)
VALUES
(101, 'Alice', 34, 'F'),
(102, 'Bob', 45, 'M'),
(103, 'Charlie', 29, 'M');

-- Output Data
SELECT s.SaleID, c.Name, p.ProductName, d.Date, l.City
FROM Sales s
JOIN Customer c ON s.CustomerID = c.CustomerID
JOIN Product p ON s.ProductID = p.ProductID
JOIN Date d ON s.DateID = d.DateID
JOIN Location l ON s.LocationID = l.LocationID;
```

**Data:**

```json
{
  "Sales": {
    "SaleID": [1, 2, 3],
    "CustomerID": [101, 102, 103],
    "ProductID": [201, 202, 203],
    "DateID": [20230101, 20230102, 20230103],
    "LocationID": [501, 502, 503],
    "TotalSalesAmount": [50.00, 75.50, 98.25],
    "QuantitySold": [2, 3, 4]
  },
  "Customer": {
    "CustomerID": [101, 102, 103],
    "Name": ["Alice", "Bob", "Charlie"],
    "Age": [34, 45, 29],
    "Gender": ["F", "M", "M"]
  }
}
```

**Flow:**

1. Identify Facts
2. Identify Dimensions
3. Design Fact Table
4. Normalize Dimensions
5. Denormalize Facts in Fact Table
6. Create Dimension Tables
7. Ensure Data Consistency

