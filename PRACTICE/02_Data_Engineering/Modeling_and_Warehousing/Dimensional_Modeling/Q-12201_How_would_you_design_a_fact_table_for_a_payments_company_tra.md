---
type: "practice"
difficulty: "Intermediate"
---

# How would you design a fact table for a payments company tracking transactions?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Fact and Dimension Keys](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Fact_and_Dimension_Keys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a fact table for a payments company, we need to focus on capturing detailed transactional data while ensuring it is optimized for analytical queries. Fact tables are typically used in star or snowflake schema designs and contain foreign keys from dimension tables along with metrics (facts). For this scenario, let's consider the key requirements: tracking transactions, which include payment amounts, dates, customer details, and various business-related dimensions.

#### Triaging the Question

The question is asking us to design a fact table for transactional data in a payments company. The interviewer might be probing our understanding of:

Data Modeling Principles: Understanding how to structure data to support reporting and analytics.
Normalization vs. Denormalization: Balancing between normalized forms (which can improve data integrity but may slow down queries) and denormalized forms (which simplify queries but can lead to redundancy).
Fact Table Design: How to construct a fact table with the right granularity, including appropriate measures and dimensions.
Step-by-Step Solution
1. Identify Key Dimensions and Measures

First, we need to identify the relevant dimensions and measures that will be stored in our fact table.

Measures:

Payment Amount (numeric)
Number of Transactions
Discount Applied
Taxes Paid

Dimensions:

Customer ID
Product ID
Date (Transaction Date, Billing Date)
Payment Method
Merchant ID
Location (City, State, Country)
2. Design the Fact Table Schema

A typical fact table for a payments company might look like this:

SQL
CREATE TABLE Transactions (
    TransactionID INT PRIMARY KEY,
    CustomerID INT NOT NULL REFERENCES Customers(CustomerID),
    ProductID INT NOT NULL REFERENCES Products(ProductID),
    PaymentMethodID INT NOT NULL REFERENCES PaymentMethods(PaymentMethodID),
    MerchantID INT NOT NULL REFERENCES Merchants(MerchantID),
    LocationID INT NOT NULL REFERENCES Locations(LocationID),
    Date DATE,
    Amount DECIMAL(10, 2) NOT NULL,
    NumberOfTransactions INT DEFAULT 1,
    DiscountApplied DECIMAL(5, 2),
    TaxesPaid DECIMAL(5, 2)
);
3. Consider the Snowflake Schema

To ensure efficient querying and adherence to best practices, we might opt for a snowflake schema. This involves breaking down one or more dimensions into multiple tables.

Customer Dimension (Flat)
SQL
CREATE TABLE Customers (
    CustomerID INT PRIMARY KEY,
    FirstName VARCHAR(50),
    LastName VARCHAR(50),
    Email VARCHAR(100)
);
Product Dimension (Flat)
SQL
CREATE TABLE Products (
    ProductID INT PRIMARY KEY,
    ProductName VARCHAR(255),
    Category VARCHAR(50)
);
Payment Method Dimension (Flat)
SQL
CREATE TABLE PaymentMethods (
    PaymentMethodID INT PRIMARY KEY,
    PaymentMethodName VARCHAR(50)
);
Merchant Dimension (Flat)
SQL
CREATE TABLE Merchants (
    MerchantID INT PRIMARY KEY,
    MerchantName VARCHAR(255),
    Category VARCHAR(50)
);
Location Dimension (Hierarchical)
SQL
CREATE TABLE Locations (
    LocationID INT PRIMARY KEY,
    City VARCHAR(100),
    State VARCHAR(50),
    Country VARCHAR(50)
);
4. Implement the Snowflake Schema in the Fact Table

We can now update our fact table to reference these dimension tables:

SQL
CREATE TABLE Transactions (
    TransactionID INT PRIMARY KEY,
    CustomerID INT NOT NULL REFERENCES Customers(CustomerID),
    ProductID INT NOT NULL REFERENCES Products(ProductID),
    PaymentMethodID INT NOT NULL REFERENCES PaymentMethods(PaymentMethodID),
    MerchantID INT NOT NULL REFERENCES Merchants(MerchantID),
    LocationID INT NOT NULL REFERENCES Locations(LocationID),
    Date DATE,
    Amount DECIMAL(10, 2) NOT NULL,
    NumberOfTransactions INT DEFAULT 1,
    DiscountApplied DECIMAL(5, 2),
    TaxesPaid DECIMAL(5, 2)
);

#### Closing Summary
and Common Mistakes

In summary, designing a fact table for a payments company involves identifying key dimensions and measures, creating appropriate tables, and potentially adopting a snowflake schema to optimize for query performance. A common mistake is over-normalizing the fact table, leading to complex queries and reduced performance.

#### Follow-Ups
Query Optimization: Discuss how partitioning and indexing can be used to improve query performance.
Data Integrity: Explain measures to ensure data integrity in a star or snowflake schema.
Scaling Considerations: Talk about potential scaling issues with large datasets and strategies for handling them.

**Comparison:**

```
Transactions
Customers
Products
PaymentMethods
Merchants
Locations
```

**Code:**

```sql
CREATE TABLE Transactions (
    TransactionID INT PRIMARY KEY,
    CustomerID INT NOT NULL REFERENCES Customers(CustomerID),
    ProductID INT NOT NULL REFERENCES Products(ProductID),
    PaymentMethodID INT NOT NULL REFERENCES PaymentMethods(PaymentMethodID),
    MerchantID INT NOT NULL REFERENCES Merchants(MerchantID),
    LocationID INT NOT NULL REFERENCES Locations(LocationID),
    Date DATE,
    Amount DECIMAL(10, 2) NOT NULL,
    NumberOfTransactions INT DEFAULT 1,
    DiscountApplied DECIMAL(5, 2),
    TaxesPaid DECIMAL(5, 2)
);

CREATE TABLE Customers (
    CustomerID INT PRIMARY KEY,
    FirstName VARCHAR(50),
    LastName VARCHAR(50),
    Email VARCHAR(100)
);

CREATE TABLE Products (
    ProductID INT PRIMARY KEY,
    ProductName VARCHAR(255),
    Category VARCHAR(50)
);

CREATE TABLE PaymentMethods (
    PaymentMethodID INT PRIMARY KEY,
    PaymentMethodName VARCHAR(50)
);

CREATE TABLE Merchants (
    MerchantID INT PRIMARY KEY,
    MerchantName VARCHAR(255),
    Category VARCHAR(50)
);

CREATE TABLE Locations (
    LocationID INT PRIMARY KEY,
    City VARCHAR(100),
    State VARCHAR(50),
    Country VARCHAR(50)
);
```

