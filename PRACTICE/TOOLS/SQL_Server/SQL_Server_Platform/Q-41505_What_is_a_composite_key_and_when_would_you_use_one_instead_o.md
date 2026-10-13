---
type: "practice"
difficulty: "Intermediate"
---

# What is a composite key and when would you use one instead of a single-column key?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ MERGE and Upserts](<../../../../TOOLS/SQL_Server/SQL_Server_Platform/MERGE_and_Upserts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A composite key in SQL Server is a combination of two or more columns that uniquely identify a record in a table. It's used when no single column can serve as a unique identifier, and it offers several advantages over using a single-column primary key.

The question essentially asks us to understand the context where you would opt for a composite key instead of a single-column key. Let’s break down the reasoning step by step:

#### Triage
Restating the Question

The core of this question revolves around when and why we use composite keys in SQL Server. The interviewer is probing our understanding of database normalization, primary keys, and how to ensure data integrity.

#### Assumptions
We are working with a relational database using SQL Server.
We need to uniquely identify records in a table.
There may be cases where no single column can achieve this uniqueness on its own.

#### First Principles Solution

Identify the Need for Uniqueness:

In a typical one-to-one relationship, a single-column key might suffice (e.g., id).
However, in scenarios with multiple related attributes that together make a record unique, we need to use a composite key.

#### Example Scenario

#### Cons
ider an OrderLineItem table where each line item is uniquely identified by the combination of order_id and product_id. Using just one column would not ensure uniqueness because several orders could have the same product, making it impossible to distinguish between them.

Creating a Composite Key:

SQL
ALTER TABLE OrderLineItem
ADD CONSTRAINT PK_OrderLineItem PRIMARY KEY (order_id, product_id);

Advantages of Using Composite Keys:

Uniqueness: Ensures each combination of columns is unique.
Normalization: Helps achieve third normal form (3NF) by removing redundancy.
Indexing: Improves query performance when the combination of columns often filters or sorts data.

Disadvantages and Trade-offs:

Larger Storage: Each row will use more space due to additional columns.
Performance Impact: While composite keys can improve certain queries, they might degrade others if not used carefully (e.g., WHERE order_id = ... AND product_id = ... vs. WHERE product_id = ...).

#### Closing Summary
and Common Mistakes
TL;DR: Use a composite key when no single column is sufficient to uniquely identify records, ensuring both uniqueness and normalization.

#### Common Mistakes
Overusing composite keys without considering performance implications.
Using too many columns in the composite key, which can lead to wasted storage.

#### Example
Input-Output
SQL
-- Create a sample table with a composite primary key
CREATE TABLE OrderLineItem (
    order_id INT,
    product_id INT,
    quantity INT,
    PRIMARY KEY (order_id, product_id)
);

-- Inserting data
INSERT INTO OrderLineItem (order_id, product_id, quantity) VALUES (101, 201, 5);
INSERT INTO OrderLineItem (order_id, product_id, quantity) VALUES (102, 202, 3);

-- Query using the composite key
SELECT * FROM OrderLineItem WHERE order_id = 101 AND product_id = 201;

**Code:**

```sql
ALTER TABLE OrderLineItem
ADD CONSTRAINT PK_OrderLineItem PRIMARY KEY (order_id, product_id);
```

**Example input / output:**

```text
-- Create a sample table with a composite primary key
CREATE TABLE OrderLineItem (
    order_id INT,
    product_id INT,
    quantity INT,
    PRIMARY KEY (order_id, product_id)
);

-- Inserting data
INSERT INTO OrderLineItem (order_id, product_id, quantity) VALUES (101, 201, 5);
INSERT INTO OrderLineItem (order_id, product_id, quantity) VALUES (102, 202, 3);

-- Query using the composite key
SELECT * FROM OrderLineItem WHERE order_id = 101 AND product_id = 201;
```

