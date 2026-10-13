---
type: "practice"
difficulty: "Basic"
---

# What is an insertion anomaly and can you give a real-world example?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Surrogate Keys in Normalization](<../../../../01_Databases_and_SQL/Database_Design/Normalization/Surrogate_Keys_in_Normalization.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

An insertion anomaly occurs in databases when data integrity issues arise during the addition of new records due to poor schema design. In this scenario, I'll explain what an insertion anomaly is and provide a real-world example using basic normalization principles.

#### Triaging the Question

The question revolves around understanding database anomalies specifically related to the process of inserting new records into a relational database. It probes your knowledge of how data integrity can be compromised due to improper table design. The key assumptions are that you understand the basics of normalization and its importance in maintaining data consistency.

#### First-Principles Solution

Normalization is the process of organizing data in a database to avoid redundancy and improve efficiency. Denormalization, on the other hand, involves sacrificing some normal form rules for performance or simplicity reasons. One common issue that arises from improper design is an insertion anomaly.

Step 1: Understanding Normalization

In a normalized database, tables are structured such that they do not contain redundant data. The most basic forms of normalization are:

First Normal Form (1NF): Each table contains atomic values.
Second Normal Form (2NF): Ensures all non-key columns depend on the whole primary key.
Third Normal Form (3NF): Eliminates transitive dependencies.
Step 2: Identifying Insertion Anomalies

An insertion anomaly occurs when it's impossible to insert data into a table due to foreign keys or other constraints, leading to incomplete records. This typically happens in partially normalized tables that have not fully followed normalization rules.

Real-World Example

#### Cons
ider an e-commerce application where you need to model products and their categories. If the schema is designed improperly, it could lead to insertion anomalies:

SQL
CREATE TABLE Products (
    ProductID INT PRIMARY KEY,
    Name VARCHAR(255),
    CategoryID INT,
    Price DECIMAL(10, 2)
);

CREATE TABLE Categories (
    CategoryID INT PRIMARY KEY,
    CategoryName VARCHAR(255)
);

In this schema:

Products and Categories are two separate tables.
A product has a foreign key to the CategoryID.

Now, let's see what happens when we try to insert data.

Attempt 1:

SQL
INSERT INTO Products (Name, CategoryID, Price) VALUES ('Smartphone', 403, 500.00);

This works fine because the category with CategoryID = 403 exists in the Categories table.

Attempt 2:

SQL
INSERT INTO Products (Name, CategoryID, Price) VALUES ('Tablet', NULL, 400.00);

Here's where things break down. The foreign key constraint on CategoryID does not allow for a null value, meaning you can't insert a product without specifying its category:

SQL
INSERT INTO Products (Name, CategoryID, Price) VALUES ('Tablet', NULL, 400.00);
-- Error: Cannot insert the value NULL into column 'CategoryID'

To resolve this issue, we need to handle insertion anomalies by ensuring that all categories are inserted first:

SQL
INSERT INTO Categories (CategoryName) VALUES ('Electronics');
INSERT INTO Products (Name, CategoryID, Price) VALUES ('Tablet', 1, 400.00);

#### Closing Summary
and Common Mistakes/Follow-Ups

An insertion anomaly occurs when the addition of a new record is impossible due to incomplete or missing references in other tables. The key to avoiding this issue is ensuring that your schema follows proper normalization rules, particularly by making sure all foreign keys are correctly handled.

#### Common mistakes
include:

Forgetting to insert related records before inserting dependent ones.
Over-normalizing too much, leading to unnecessary complexity and reducing performance.

If the interviewer asks follow-ups, they might delve into:

How to manage cascading inserts in a transaction-safe manner.
Strategies for denormalization when performance is critical.
Handling edge cases where some data might not have categories initially.

**Comparison:**

| Table        | Columns      |
|--------------|-------------|
| Products     | ProductID, Name, CategoryID, Price |
| Categories   | CategoryID, CategoryName           |

**Code:**

```sql
CREATE TABLE Products (
    ProductID INT PRIMARY KEY,
    Name VARCHAR(255),
    CategoryID INT,
    Price DECIMAL(10, 2)
);

CREATE TABLE Categories (
    CategoryID INT PRIMARY KEY,
    CategoryName VARCHAR(255)
);
```

**Example input / output:**

```text
Attempt 1:
INSERT INTO Products (Name, CategoryID, Price) VALUES ('Smartphone', 403, 500.00);
-- Success

Attempt 2:
INSERT INTO Products (Name, CategoryID, Price) VALUES ('Tablet', NULL, 400.00); -- Error: Cannot insert the value NULL into column 'CategoryID'
```

**Data:**

```sql
INSERT INTO Products (Name, CategoryID, Price) VALUES ('Smartphone', 403, 500.00);
INSERT INTO Products (Name, CategoryID, Price) VALUES ('Tablet', NULL, 400.00); -- Error: Cannot insert the value NULL into column 'CategoryID'
```

**Flow:**

1. Yes
2. No
3. Insert Product
4. Does the category exist?
5. (Success)
6. Insert Category First
7. Then Insert Product

