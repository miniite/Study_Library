---
type: "practice"
difficulty: "Basic"
---

# Explain the difference between a fact table and a dimension table with an example.

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Fact Dimension Integrity](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Data_Warehousing/Fact_Dimension_Integrity.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In data warehousing, fact tables and dimension tables serve different purposes and together form a star or snowflake schema. A fact table contains quantitative measures (facts) about business events, while dimension tables store descriptive information (dimensions) that help analyze these facts.

#### Triage

The question asks to explain the difference between a fact table and a dimension table with an example. The key concepts involve understanding the structure of data warehouses, specifically how fact and dimension tables work together in analyzing business data.

#### First-Principles Solution
Fact Table

A fact table is central to the data warehouse schema and contains quantitative information about business events. For example, a sales transaction can be recorded with metrics such as SalesAmount, QuantitySold, etc. Fact tables typically have foreign keys linking to dimension tables.

#### Example
SalesFact

SQL
CREATE TABLE SalesFact (
    SaleID INT PRIMARY KEY,
    ProductID INT,
    CustomerID INT,
    StoreID INT,
    DateID INT,
    QuantitySold INT,
    SalesAmount DECIMAL(10, 2)
);
Dimension Table

A dimension table contains qualitative attributes that describe the fact table. Dimensions are usually smaller than fact tables and provide context for the facts.

#### Example
Product

SQL
CREATE TABLE Product (
    ProductID INT PRIMARY KEY,
    ProductName VARCHAR(100),
    Category VARCHAR(50)
);

#### Example
Customer

SQL
CREATE TABLE Customer (
    CustomerID INT PRIMARY KEY,
    FirstName VARCHAR(50),
    LastName VARCHAR(50),
    Country VARCHAR(50)
);

#### Closing Summary

Fact tables and dimension tables are fundamental components of data warehousing. Fact tables store the core metrics, while dimension tables provide context by capturing additional details. Together, they enable comprehensive analysis and reporting.

#### Common Mistakes/Follow-Ups
Mixing Up Fact and Dimension: Confusing fact and dimension tables can lead to incorrect schema design.
Lack of Foreign Keys: Forgetting to link fact tables with dimensions through foreign keys might result in a disconnected data model.
Overcomplication of Dimensions: Overly complex dimension tables without clear context can complicate analysis.

**Comparison:**

```
Fact Table Example	Dimension Table Examples
SalesFact - Quantitative metrics about sales transactions.	Product - Describes products sold, e.g., name and category.
Customer - Contains customer details for each transaction.	Store - Information about the store where a sale occurred.
```

**Code:**

```sql
CREATE TABLE SalesFact (
    SaleID INT PRIMARY KEY,
    ProductID INT,
    CustomerID INT,
    StoreID INT,
    DateID INT,
    QuantitySold INT,
    SalesAmount DECIMAL(10, 2)
);

CREATE TABLE Product (
    ProductID INT PRIMARY KEY,
    ProductName VARCHAR(100),
    Category VARCHAR(50)
);

CREATE TABLE Customer (
    CustomerID INT PRIMARY KEY,
    FirstName VARCHAR(50),
    LastName VARCHAR(50),
    Country VARCHAR(50)
);
```

