---
type: "practice"
difficulty: "Basic"
---

# Why are fact tables described as 'tall and narrow' and dimensions as 'short and wide'?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Fact and Dimension Keys](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Fact_and_Dimension_Keys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In data modeling, particularly in dimensional modeling, fact tables are often referred to as "tall and narrow," while dimensions are known as "short and wide." This characterization is rooted in their structure and how they are used within a data warehouse or analytics environment.

#### Triage

The question probes the core principles behind the design of fact tables and dimensions. We need to understand why these terms are used, what benefits accrue from this layout, and any potential trade-offs.

#### First-Principles Solution
Why Fact Tables Are Described as "Tall and Narrow"
Granularity: Fact tables capture detailed transactional data with a high level of granularity. Each row in the fact table represents a specific event or transaction.
Volume of Data: Due to their granular nature, fact tables can contain millions or even billions of rows, making them "tall." They are also optimized for queries that need to analyze these detailed records efficiently.
Why Dimensions Are Described as "Short and Wide"
Attributes vs. Facts: Dimensions store descriptive attributes about the facts in the fact table. These attributes might include time (day, month, year), location (city, state, country), or product details (brand, model).
Aggregation: Dimensions are typically not used for complex calculations; instead, they provide context to the data stored in fact tables.
Width: Since dimensions contain many descriptive attributes, they can have a wide schema with numerous columns.
Detailed Explanation
Fact Tables as "Tall and Narrow"
Granularity: Consider an online retail scenario where each transaction is logged individually. Each row in the fact table would represent one sale, containing information like product ID, customer ID, price, date of purchase, etc.
Volume: If a retailer has millions of customers making purchases daily, their fact table will be extremely tall, with each sale being a distinct record.
SQL
CREATE TABLE Sales (
    SaleID INT PRIMARY KEY,
    ProductID INT NOT NULL,
    CustomerID INT NOT NULL,
    Price DECIMAL(10, 2) NOT NULL,
    PurchaseDate DATE NOT NULL,
    -- More attributes...
);
Dimensions as "Short and Wide"
Attributes: The dimension table for customers might include details like name, address, city, state, country, etc., making it wide.
Aggregation: While the fact table tracks specific sales, the customer dimension provides context to understand who made these purchases.
SQL
CREATE TABLE Customers (
    CustomerID INT PRIMARY KEY,
    FirstName VARCHAR(50) NOT NULL,
    LastName VARCHAR(50) NOT NULL,
    Address1 VARCHAR(255),
    Address2 VARCHAR(255),
    City VARCHAR(50),
    State VARCHAR(50),
    Country VARCHAR(50),
    -- More attributes...
);

#### Trade-Offs
and Considerations
Join Operations: Since fact tables are tall, joining them with wide dimensions can be resource-intensive. Optimizing these joins is crucial for performance.
Storage Efficiency: The narrowness of fact tables and the width of dimensions helps manage storage more efficiently.

#### Closing Summary

In summary, fact tables in a star or snowflake schema are designed to be "tall and narrow" due to their high granularity and volume of detailed data. Dimensions, on the other hand, are structured as "short and wide" to store a variety of descriptive attributes that provide context to the fact table.

#### Common Mistakes
and Follow-Ups
Incorrect Schema Design: Failing to properly design dimensions can lead to poor query performance.
Ignoring Performance Impact: Not considering the impact of join operations between tall fact tables and wide dimensions can result in suboptimal data models.

#### Follow-Up
Questions:
How do you optimize joins between fact and dimension tables?
What are some best practices for designing a dimensional model?

**Comparison:**

```
Sales
CustomerID
ProductID
Price
PurchaseDate
Customers
CustomerID
FirstName
LastName
Address1
Address2
City
State
Country
```

**Code:**

```sql
-- Example Fact Table (Sales)
CREATE TABLE Sales (
    SaleID INT PRIMARY KEY,
    ProductID INT NOT NULL,
    CustomerID INT NOT NULL,
    Price DECIMAL(10, 2) NOT NULL,
    PurchaseDate DATE NOT NULL
);

-- Example Dimension Table (Customers)
CREATE TABLE Customers (
    CustomerID INT PRIMARY KEY,
    FirstName VARCHAR(50),
    LastName VARCHAR(50),
    Address1 VARCHAR(255),
    Address2 VARCHAR(255),
    City VARCHAR(50),
    State VARCHAR(50),
    Country VARCHAR(50)
);
```

**Example input / output:**

```text
Input:
SELECT T1.CustomerID, T2.FirstName, T2.LastName, SUM(T1.Price)
FROM Sales AS T1
JOIN Customers AS T2 ON T1.CustomerID = T2.CustomerID
GROUP BY T1.CustomerID, T2.FirstName, T2.LastName;

Output:
+------------+-----------+-----------+------------------+
| CustomerID | FirstName | LastName  | TotalSpent       |
+------------+-----------+-----------+------------------+
| 2001       | John      | Doe       | 99.99            |
| 2002       | Jane      | Smith     | 149.99           |
+------------+-----------+-----------+------------------+
```

**Data:**

```json
{
  "Sales": [
    {
      "SaleID": 1,
      "ProductID": 101,
      "CustomerID": 2001,
      "Price": 99.99,
      "PurchaseDate": "2023-10-01"
    },
    {
      "SaleID": 2,
      "ProductID": 102,
      "CustomerID": 2002,
      "Price": 149.99,
      "PurchaseDate": "2023-10-02"
    }
  ],
  "Customers": [
    {
      "CustomerID": 2001,
      "FirstName": "John",
      "LastName": "Doe",
      "Address1": "123 Main St",
      "City": "New York",
      "State": "NY",
      "Country": "USA"
    },
    {
      "CustomerID": 2002,
      "FirstName": "Jane",
      "LastName": "Smith",
      "Address1": "456 Elm St",
      "City": "Los Angeles",
      "State": "CA",
      "Country": "USA"
    }
  ]
}
```

**Flow:**

1. Query Fact Table
2. Join with Dimensions
3. Compute Aggregates
4. Return Results

