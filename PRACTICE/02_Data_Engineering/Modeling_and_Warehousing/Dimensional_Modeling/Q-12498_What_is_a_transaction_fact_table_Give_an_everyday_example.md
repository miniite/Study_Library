---
type: "practice"
difficulty: "Basic"
---

# What is a transaction fact table? Give an everyday example.

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Transaction vs Snapshot Facts](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Transaction_vs_Snapshot_Facts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A transaction fact table in data modeling, particularly within star and snowflake schemas, records specific events or transactions over time. It captures granular details like quantities, prices, timestamps, etc., associated with business activities. Think of it as a detailed log where each row represents a transaction.

#### Triaging the Question

The question asks for an understanding of what a transaction fact table is and to provide an everyday example. The interviewer might be looking to understand your grasp of data modeling fundamentals, specifically in how facts are stored and related to dimensions in star or snowflake schemas.

#### First-Principles Solution
Definition: A transaction fact table contains measurements or metrics that describe events or transactions. For instance, it could record the quantity of items sold at a specific store on a particular day.
Structure: It is linked with dimension tables that provide context to these facts, such as date, product details, and customer information.

#### Example
Consider a retail scenario where you track sales transactions.
Detailed Explanation

#### Example
Sales Transaction Table

Imagine a retail store that sells various products across different locations. Each time a sale is made, it generates detailed transaction records. A typical transaction fact table might look like this:

SaleID	ProductID	StoreID	QuantitySold	PricePerUnit	DateOfSale
101	2	5	3	4.99	2023-10-01
102	1	7	1	6.99	2023-10-02
103	2	5	2	4.99	2023-10-03

Each row here is a transaction, capturing details like the quantity sold (QuantitySold), price per unit (PricePerUnit), and date of sale (DateOfSale). This table could be linked with dimension tables for more context:

Product Dimension: Stores information about each product (ID, Name, Category).
Store Dimension: Stores information about each store (ID, Location, Manager).
Time Dimension: Stores dates and other time-related details.
Why Use a Fact Table?

Fact tables are crucial because they allow for detailed analysis. For example, you could easily calculate total sales by product or store using these tables along with their associated dimensions.

#### Closing Summary

A transaction fact table is essential in data modeling as it records the granular details of specific events or transactions. By linking them to appropriate dimension tables, analysts can gain deeper insights and perform complex analyses on the business activities recorded.

#### Common Mistakes
and Follow-Ups
Mistake: Failing to recognize that a transaction fact table must contain detailed metrics.

#### Follow-Up
Interviewers might ask about how different types of dimensions (like time or location) interact with transaction tables, or about designing a schema for such a system.

**Comparison:**

```
SaleID	ProductID	StoreID	QuantitySold	PricePerUnit	DateOfSale
101	2	5	3	4.99	2023-10-01
102	1	7	1	6.99	2023-10-02
103	2	5	2	4.99	2023-10-03
ProductID	Name	Category
1	Laptop	Electronics
2	Phone	Electronics
StoreID	Location	Manager
5	New York	John Doe
7	Los Angeles	Jane Smith
```

**Code:**

```sql
-- Example SQL Schema for the Fact Table

CREATE TABLE SalesTransaction (
    SaleID INT PRIMARY KEY,
    ProductID INT,
    StoreID INT,
    QuantitySold INT,
    PricePerUnit DECIMAL(10,2),
    DateOfSale DATE
);

-- Example Dimension Tables
CREATE TABLE Products (
    ProductID INT PRIMARY KEY,
    Name VARCHAR(100),
    Category VARCHAR(50)
);

CREATE TABLE Stores (
    StoreID INT PRIMARY KEY,
    Location VARCHAR(100),
    Manager VARCHAR(100)
);
```

**Example input / output:**

```text
| SaleID | ProductID | StoreID | QuantitySold | PricePerUnit | DateOfSale   |
|--------|-----------|---------|--------------|--------------|--------------|
| 101    | 2         | 5       | 3            | 4.99         | 2023-10-01  |
| 102    | 1         | 7       | 1            | 6.99         | 2023-10-02  |
| 103    | 2         | 5       | 2            | 4.99         | 2023-10-03  |

- **SaleID**: Unique identifier for each sale.
- **ProductID**: Reference to the product sold.
- **StoreID**: Reference to the store where the sale occurred.
- **QuantitySold**: Number of items sold in this transaction.
- **PricePerUnit**: Price at which one unit was sold.
- **DateOfSale**: Date and time when the sale took place.

This setup allows for easy aggregation, such as summing up total sales by product or store over different dates.
```

**Data:**

```json
{
  "SalesTransaction": [
    {
      "SaleID": 101,
      "ProductID": 2,
      "StoreID": 5,
      "QuantitySold": 3,
      "PricePerUnit": 4.99,
      "DateOfSale": "2023-10-01"
    },
    {
      "SaleID": 102,
      "ProductID": 1,
      "StoreID": 7,
      "QuantitySold": 1,
      "PricePerUnit": 6.99,
      "DateOfSale": "2023-10-02"
    },
    {
      "SaleID": 103,
      "ProductID": 2,
      "StoreID": 5,
      "QuantitySold": 2,
      "PricePerUnit": 4.99,
      "DateOfSale": "2023-10-03"
    }
  ],
  "Products": [
    {
      "ProductID": 1,
      "Name": "Laptop",
      "Category": "Electronics"
    },
    {
      "ProductID": 2,
      "Name": "Phone",
      "Category": "Electronics"
    }
  ],
  "Stores": [
    {
      "StoreID": 5,
      "Location": "New York",
      "Manager": "John Doe"
    },
    {
      "StoreID": 7,
      "Location": "Los Angeles",
      "Manager": "Jane Smith"
    }
  ]
}
```

**Flow:**

1. Sales Transaction Table
2. Product Dimension
3. Store Dimension
4. Time Dimension

