---
type: "practice"
difficulty: "Basic"
---

# What is the difference between a fact table and a dimension table?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Fact and Dimension Keys](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Fact_and_Dimension_Keys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand the differences between a fact table and a dimension table, it's essential to first grasp their roles in data modeling, particularly within the context of Star and Snowflake schemas.

#### Triage
What is the Question Really Asking?

The question essentially probes your understanding of how star and snowflake schemas are structured, with specific focus on the distinct characteristics of fact tables and dimension tables. In a star schema, both types of tables are present, while in a snowflake schema, dimension tables can be further normalized.

Core Concept: Fact Table vs Dimension Table

A fact table contains the main metrics or measurements about which you collect data. It is typically wide, with many columns representing different fact measures and fewer rows, as it captures multiple instances of these facts over time. On the other hand, a dimension table provides context to the fact table by describing attributes (often called dimensions) that help in analyzing the data within the fact table.

Step-by-Step Solution from First Principles

Definition of Fact Table:

The fact table contains quantitative data or metrics. Think of it as the core table where all your numerical information resides.

#### Example
In a sales database, a fact table might contain columns like SalesID, ProductID, QuantitySold, and TotalAmount.

Definition of Dimension Table:

The dimension table provides qualitative data that helps in understanding the context of the facts. It typically contains many rows but fewer columns.

#### Example
A related dimension table might have columns like ProductID, ProductName, CategoryName, BrandName.

Relationship Between Fact and Dimension Tables:

Fact tables are usually linked to one or more dimension tables via foreign keys. This linkage allows for detailed analysis of the data.
For instance, in our example, the fact table would reference the ProductID from the dimension table.

Star Schema vs Snowflake Schema:

In a star schema, all fact and dimension tables are directly related to the fact table, forming a simple, easily understandable structure.
A snowflake schema normalizes some of the dimension tables, leading to a more complex but potentially cleaner data model. This can be beneficial in reducing data redundancy.

#### Closing Summary
and Common Mistakes

To summarize, the key difference is that fact tables store quantitative measures while dimension tables provide qualitative context. Understanding these roles correctly is crucial for designing efficient and meaningful database schemas. A common mistake is not properly normalizing dimensions within a snowflake schema, leading to potential performance issues or complicating queries unnecessarily.

#### Common Mistakes/Follow-Ups
Mistake 1: Confusing the roles of fact tables and dimension tables.
Mistake 2: Over-normalizing in a star schema where it's not necessary.

#### Follow-Up
Discussing scenarios where you might prefer one over the other, such as when dealing with very large datasets or needing complex queries.

**Comparison:**

| Fact Table (SalesFact) | Dimension Table (ProductsDim) |
|------------------------|------------------------------|
| SalesID                | ProductID                     |
| QuantitySold           | ProductName                   |
| TotalAmount            | CategoryName                  |
| OrderDate              | BrandName                     |

**Code:**

```sql
-- Example of Fact Table Creation
CREATE TABLE SalesFact (
    SalesID INT PRIMARY KEY,
    ProductID INT NOT NULL,
    QuantitySold INT,
    TotalAmount DECIMAL(10, 2),
    OrderDate DATE
);

-- Example of Dimension Table Creation
CREATE TABLE ProductsDim (
    ProductID INT PRIMARY KEY,
    ProductName VARCHAR(50),
    CategoryName VARCHAR(30),
    BrandName VARCHAR(20)
);
```

**Example input / output:**

```text
Input:
Fact Table (SalesFact)
Output:
[
  {"SalesID": 1, "ProductID": 101, "QuantitySold": 15, "TotalAmount": 375.00},
  {"SalesID": 2, "ProductID": 102, "QuantitySold": 8, "TotalAmount": 160.00}
]

Input:
Dimension Table (ProductsDim)
Output:
[
  {"ProductID": 101, "ProductName": "Widget A", "CategoryName": "Electronics", "BrandName": "TechPro"},
  {"ProductID": 102, "ProductName": "Gadget B", "CategoryName": "Electronics", "BrandName": "GadgetCo"}
]
```

**Data:**

```json
{
  "SalesFact": [
    {
      "SalesID": 1,
      "ProductID": 101,
      "QuantitySold": 15,
      "TotalAmount": 375.00,
      "OrderDate": "2023-06-01"
    },
    {
      "SalesID": 2,
      "ProductID": 102,
      "QuantitySold": 8,
      "TotalAmount": 160.00,
      "OrderDate": "2023-06-05"
    }
  ],
  "ProductsDim": [
    {
      "ProductID": 101,
      "ProductName": "Widget A",
      "CategoryName": "Electronics",
      "BrandName": "TechPro"
    },
    {
      "ProductID": 102,
      "ProductName": "Gadget B",
      "CategoryName": "Electronics",
      "BrandName": "GadgetCo"
    }
  ]
}
```

**Flow:**

1. Fact Table
2. Dimension Tables
3. SalesFact Table
4. Products Dim Table
5. Order Date
6. Product Name
7. Category Name
8. Brand Name

