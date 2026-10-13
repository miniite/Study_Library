---
type: "practice"
difficulty: "Deep"
---

# What is a bridge table and how would you model a many-to-many relationship between accounts and products in a star schema?

**Difficulty:** Deep · **Levels:** Fresher · [↑ Relationship Modeling](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Relationship_Modeling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

A bridge table, also known as an intersection or association table, is used to manage relationships that are not one-to-one, one-to-many, or many-to-one. In the context of a star schema, which is commonly used in data warehousing for query optimization and performance, we need to model complex relationships between entities like accounts and products. A bridge table can help us establish a many-to-many relationship without violating the normalization rules.

#### Triaging the Question

The question asks about modeling a many-to-many relationship between accounts and products using a star schema. Here’s what the interviewer is probing:

Understanding of Entity-Relationship Diagrams (ERDs) and Normalization.
Knowledge of Star Schema Design Principles.
Ability to Identify and Implement Solutions for Complex Relationships.
Step-by-Step Solution
1. Understanding Many-to-Many Relationships

In a many-to-many relationship, an entity can be related to multiple instances of another entity, and vice versa. For example, an account can have multiple products, and a product can be used by multiple accounts.

2. Traditional Table Design (Without Bridge Tables)

If we were not using a star schema, we might initially design the database as follows:

accounts table: account_id, account_name
products table: product_id, product_name

Since many-to-many relationships are common in real-world scenarios, this would require an intermediate join table. However, in a star schema, direct joins between fact tables and dimension tables should be minimized to improve query performance.

3. Star Schema and Fact Tables

In a star schema, the central fact table contains aggregated data that is related to multiple dimensions through bridge tables or lookup tables. For example:

sales_fact table: account_id, product_id, quantity_sold, date

Here, both accounts and products are dimension tables.

4. Implementing the Many-to-Many Relationship

To model a many-to-many relationship between accounts and products in a star schema:

Create a Bridge Table: This table will act as an intermediary to link accounts with products.
Dimension Tables: Ensure that both accounts and products are normalized dimension tables.

Let’s assume the following schema for simplicity:

SQL
CREATE TABLE accounts (
    account_id INT PRIMARY KEY,
    account_name VARCHAR(100)
);

CREATE TABLE products (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100)
);

CREATE TABLE account_products (  -- Bridge Table
    account_product_id INT AUTO_INCREMENT PRIMARY KEY,
    account_id INT NOT NULL,
    product_id INT NOT NULL,
    FOREIGN KEY (account_id) REFERENCES accounts(account_id),
    FOREIGN KEY (product_id) REFERENCES products(product_id)
);

This bridge table account_products ensures that an account can have many products and a product can be associated with many accounts.

5. Querying the Data

In a star schema, when you want to retrieve data related to both accounts and their associated products, your query might look something like this:

SQL
SELECT
    a.account_name,
    p.product_name,
    ap.quantity_sold
FROM
    sales_fact sf
JOIN
    account_products ap ON sf.account_id = ap.account_id AND sf.product_id = ap.product_id
JOIN
    accounts a ON ap.account_id = a.account_id
JOIN
    products p ON ap.product_id = p.product_id;

#### Closing Summary
and Common Mistakes/Follow-Ups
Common Mistake 1: Forgetting to use the bridge table, resulting in direct joins between fact tables.
Common Mistake 2: Not considering normalization rules while designing dimension tables.

#### Follow-Up
Question: How would you optimize this query for better performance if there are millions of records?
Triaging and Reassuring

By using a bridge table, we ensure that the many-to-many relationship is properly modeled in a star schema. This approach avoids denormalizing fact tables while maintaining data integrity and performance.

**Code:**

```sql
CREATE TABLE accounts (
    account_id INT PRIMARY KEY,
    account_name VARCHAR(100)
);

CREATE TABLE products (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100)
);

CREATE TABLE account_products (  -- Bridge Table
    account_product_id INT AUTO_INCREMENT PRIMARY KEY,
    account_id INT NOT NULL,
    product_id INT NOT NULL,
    FOREIGN KEY (account_id) REFERENCES accounts(account_id),
    FOREIGN KEY (product_id) REFERENCES products(product_id)
);
```

