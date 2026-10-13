---
type: "practice"
difficulty: "Basic"
---

# What is a candidate key and how does it relate to the primary key you finally choose?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Constraint Definitions](<../../../../TOOLS/MySQL/MySQL_Platform/Constraint_Definitions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In database development, particularly with MySQL, understanding the concepts of keys—especially candidate keys and primary keys—is crucial for ensuring data integrity and efficient data retrieval. A candidate key is any attribute or combination of attributes in a table that can uniquely identify each record in that table. On the other hand, a primary key is a specific type of candidate key chosen to serve as the unique identifier for rows in a table.

#### Triage

The question asks us to explain what a candidate key is and how it relates to the primary key we choose. The interviewer is probing your understanding of relational database design principles, specifically how keys contribute to data consistency and integrity. We will assume that you are familiar with basic SQL and MySQL syntax for defining tables.

First Principles

Firstly, let’s define what a candidate key is:

Candidate Key: Any column or combination of columns in a table that can uniquely identify each row without any NULL values.
This means that no two rows in the table will have the same value(s) for these columns.

Next, we'll look at how this relates to the primary key. The primary key is one specific candidate key chosen from among all possible candidates:

Primary Key: A column or combination of columns selected as the main identifier for each row in a database table.
It must uniquely identify every row and cannot contain NULL values.
Step-by-Step Solution
Identifying Candidate Keys:
In a Customers table, possible candidate keys could be CustomerID, Email, or Name + Surname. However, Email might not be unique if multiple customers use the same email address.
Selecting the Primary Key:
From all candidate keys, we select one that meets specific criteria such as being unique and not containing NULL values. For instance, in a typical scenario, CustomerID would be chosen as the primary key because it uniquely identifies each customer.

#### Example

Let's say you have a table called Orders. Possible candidate keys include:

OrderNumber (could repeat if multiple orders have the same number)
CustomerID + OrderDate (unique but may contain NULLs for different reasons)

You would choose OrderNumber as the primary key because it uniquely identifies each row and doesn't allow NULL values.

#### Closing Summary

In summary, a candidate key is any attribute that can uniquely identify a record in a table. The primary key is a specific candidate key chosen to ensure uniqueness and non-nullability for efficient data management. Understanding these concepts helps in designing robust database schemas.

#### Common Mistakes/Follow-Ups
Common Mistake: Choosing multiple columns as the primary key when a single column can suffice.

#### Follow-Up
Discuss scenarios where composite keys (multiple columns) might be necessary, such as OrderNumber + ProductID to uniquely identify specific items in an order.

**Comparison:**

| OrderNumber | CustomerID | OrderDate       | ProductID | Quantity |
|-------------|------------|----------------|-----------|----------|
| 67890       | 54321      | 2023-02-10     | 102       | 1        |

**Code:**

```sql
-- Example of creating a table with candidate and primary keys
CREATE TABLE Orders (
    OrderNumber INT PRIMARY KEY,
    CustomerID INT NOT NULL,
    OrderDate DATE NOT NULL,
    ProductID INT NOT NULL,
    Quantity INT CHECK (Quantity > 0)
);
```

**Example input / output:**

```text
-- Inserting a new order
INSERT INTO Orders (OrderNumber, CustomerID, OrderDate, ProductID, Quantity)
VALUES (67890, 54321, '2023-02-10', 102, 1);

SELECT * FROM Orders WHERE OrderNumber = 67890;
```

**Data:**

```json
{
    "OrderNumber": 12345,
    "CustomerID": 98765,
    "OrderDate": "2023-01-15",
    "ProductID": 101,
    "Quantity": 2
}
```

**Flow:**

1. Yes
2. No
3. Identify Candidate Keys
4. Is each unique?
5. Choose Primary Key
6. Find alternative unique key oradd a surrogate key likeOrderNumber
7. Ensure non-nullability anduniqueness

