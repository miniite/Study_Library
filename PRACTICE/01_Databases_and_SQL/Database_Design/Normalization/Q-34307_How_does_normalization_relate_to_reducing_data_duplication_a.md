---
type: "practice"
difficulty: "Basic"
---

# How does normalization relate to reducing data duplication across a database?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Normalization Fundamentals](<../../../../01_Databases_and_SQL/Database_Design/Normalization/Normalization_Fundamentals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Normalization is crucial in database design to ensure data integrity and reduce redundancy. At its core, normalization means organizing your database into tables with simple structures that are easy to understand and maintain. The primary goal of normalization is to eliminate redundant data by storing each piece of information exactly once.

To understand how normalization reduces data duplication across a database, we need to delve into the basics of normal forms (1NF, 2NF, 3NF). Let's break it down step-by-step from first principles:

First, denormalization is often mentioned in contrast with normalization. Denormalization intentionally reintroduces redundancy for performance reasons or because of business requirements. However, normalization addresses the root cause of data duplication by breaking down large tables into smaller ones and ensuring that each table has a single key.

Normalization Steps
First Normal Form (1NF): Ensures that each column in a table contains atomic (indivisible) values.
Second Normal Form (2NF): Builds on 1NF by eliminating partial dependencies, meaning non-key columns should be fully dependent on the primary key.
Third Normal Form (3NF): Further refines 2NF by ensuring that there are no transitive dependencies; a column is independent of the primary key if it depends only on another non-key.

For example, consider a scenario where you have an Orders table with columns for OrderID, CustomerID, and Product. Initially, this might be denormalized to store multiple product entries per order. However, normalizing this would involve splitting the data into separate tables: one for customers and another for products.

#### Example Scenario

Suppose we have an Orders table with the following structure:

OrderID	CustomerID	Product1	Quantity1	Product2	Quantity2
101	CUST-1	PROD-A	5	PROD-B	3

Normalization would involve splitting this table into multiple tables:

Customers Table:

SQL
CREATE TABLE Customers (
    CustomerID VARCHAR PRIMARY KEY,
    FirstName VARCHAR,
    LastName VARCHAR,
    Email VARCHAR
);

Products Table:

SQL
CREATE TABLE Products (
    ProductID INT AUTO_INCREMENT PRIMARY KEY,
    Name VARCHAR,
    Price DECIMAL(10, 2)
);

Orders Table:

SQL
CREATE TABLE Orders (
    OrderID INT AUTO_INCREMENT PRIMARY KEY,
    CustomerID VARCHAR,
    OrderDate DATE,
    FOREIGN KEY (CustomerID) REFERENCES Customers(CustomerID)
);

OrderDetails Table:

SQL
CREATE TABLE OrderDetails (
    OrderDetailID INT AUTO_INCREMENT PRIMARY KEY,
    OrderID INT,
    ProductID INT,
    Quantity INT,
    FOREIGN KEY (OrderID) REFERENCES Orders(OrderID),
    FOREIGN KEY (ProductID) REFERENCES Products(ProductID)
);
Benefits of Normalization
Data Integrity: Reduces the risk of data inconsistencies.
Efficiency: Simplifies queries and updates, making them faster and more efficient.
Maintainability: Easier to maintain and extend as the database grows.

#### Common Mistakes
Over-normalization: Going too far in normalization can lead to excessive tables and joins, which can degrade performance.
Ignoring Business Requirements: Sometimes denormalizing for performance reasons is necessary, especially in real-time applications where read speed is critical.
Inadequate Indexing: Proper indexing is crucial after normalizing, as it affects query performance.

#### Conclusion

Normalization helps reduce data duplication by organizing the database into smaller, more manageable tables with simple relationships. While denormalization can be used for specific use cases to improve performance, normalization should be the default approach in most scenarios to ensure data integrity and efficiency.

**Code:**

```sql
-- Example SQL Create Statements
CREATE TABLE Customers (
    CustomerID VARCHAR PRIMARY KEY,
    FirstName VARCHAR,
    LastName VARCHAR,
    Email VARCHAR
);

CREATE TABLE Products (
    ProductID INT AUTO_INCREMENT PRIMARY KEY,
    Name VARCHAR,
    Price DECIMAL(10, 2)
);

CREATE TABLE Orders (
    OrderID INT AUTO_INCREMENT PRIMARY KEY,
    CustomerID VARCHAR,
    OrderDate DATE,
    FOREIGN KEY (CustomerID) REFERENCES Customers(CustomerID)
);

CREATE TABLE OrderDetails (
    OrderDetailID INT AUTO_INCREMENT PRIMARY KEY,
    OrderID INT,
    ProductID INT,
    Quantity INT,
    FOREIGN KEY (OrderID) REFERENCES Orders(OrderID),
    FOREIGN KEY (ProductID) REFERENCES Products(ProductID)
);
```

