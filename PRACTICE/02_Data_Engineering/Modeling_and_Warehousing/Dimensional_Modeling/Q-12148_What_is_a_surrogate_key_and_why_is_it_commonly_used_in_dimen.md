---
type: "practice"
difficulty: "Basic"
---

# What is a surrogate key and why is it commonly used in dimension tables?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Date Dimensions](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Date_Dimensions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In data modeling, especially within dimensional databases like star or snowflake schemas, a surrogate key is an artificial primary key that replaces natural keys. Surrogate keys are typically integers and have no inherent meaning related to the business domain but serve as unique identifiers for records in tables.

#### Triaging the Question

The question asks about the concept of surrogate keys and their common use in dimension tables. The interviewer might be probing your understanding of:

What a surrogate key is.
Why natural keys are not always ideal.
The benefits and trade-offs of using surrogate keys.
How surrogate keys fit into data modeling practices, particularly in star/snowflake schemas.

#### First-Principles Solution

A surrogate key is often used as the primary key in a dimension table because:

Stability: Surrogate keys are immutable and do not change over time or with business changes.
Performance: They allow for faster join operations, especially when dealing with large datasets.
Normalization: Surrogate keys help enforce referential integrity without exposing sensitive information from the natural key.

In a star schema, fact tables contain the foreign keys linking to dimension tables, which use surrogate keys as their primary keys. This structure is designed to facilitate efficient querying and reporting by separating slowly changing dimensions (SCDs) and bridging facts with dimensions through stable identifiers.

#### Example Scenario

Imagine a retail company tracking sales data. A typical star schema might include:

Fact table: Sales

Foreign key: ProductID (surrogate key from the Products dimension table)
Foreign key: CustomerID (surrogate key from the Customers dimension table)
Foreign key: DateID (surrogate key from the Dates dimension table)

Dimension tables:

Products
Surrogate key: ProductID
Natural keys: ProductName, ProductCode
Customers
Surrogate key: CustomerID
Natural keys: CustomerName, CustomerNumber

#### Closing Summary

In summary, surrogate keys are used in dimension tables to provide a stable and performant identifier. They help in maintaining the integrity of the data model by abstracting away the complexities of natural keys, such as changes or updates that could affect query performance.

#### Common Mistakes/Follow-Ups
Confusion with Natural Keys: Often, interviewers might ask you to differentiate between surrogate and natural keys.
Understanding Performance Implications: You may be asked how surrogate keys impact join operations in the star schema.
Handling Changes in Data Models: They might explore scenarios where a dimension table’s structure changes, such as adding new attributes or modifying existing ones.

**Comparison:**

| Dimension Table: Products        | Dimension Table: Dates          |
|----------------------------------|--------------------------------|
| **ProductID** | **ProductName**  | **DateID** | **Date**        |
| 1              | Laptop           | 501        | 2023-10-01      |
| 2              | Smartphone       | 502        | 2023-10-02      |

**Code:**

```sql
-- Example of creating a surrogate key in SQL
CREATE TABLE Products (
    ProductID INT PRIMARY KEY,  -- Surrogate Key
    ProductName VARCHAR(100),
    ProductCode VARCHAR(20)
);

CREATE TABLE Sales (
    SaleID INT PRIMARY KEY,
    ProductID INT,  -- Foreign key linking to Products
    DateID INT,     -- Another foreign key for Dates table
    Quantity INT,
    FOREIGN KEY (ProductID) REFERENCES Products(ProductID)
);
```

**Example input / output:**

| Input | Output |
|-------|--------|
| Sales table | Product ID (surrogate key) and Date ID (foreign key from Dates dimension table) linked to corresponding surrogate keys in Products and Dates tables. |

**Data:**

```json
{
  "Products": [
    { "ProductID": 1, "ProductName": "Laptop", "ProductCode": "LP001" },
    { "ProductID": 2, "ProductName": "Smartphone", "ProductCode": "SP001" }
  ],
  "Sales": [
    { "SaleID": 1, "ProductID": 1, "DateID": 501 },
    { "SaleID": 2, "ProductID": 2, "DateID": 502 }
  ]
}
```

**Flow:**

1. Sales Table
2. ProductID
3. Products Dimension Table
4. DateID
5. Dates Dimension Table

