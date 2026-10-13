---
type: "practice"
difficulty: "Intermediate"
---

# What is referential integrity and how is it enforced through the schema?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Polymorphic Associations](<../../../../01_Databases_and_SQL/Database_Design/Schema_Design/Polymorphic_Associations.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

Referential integrity ensures that relationships between tables in a database are consistent, preventing orphaned records by enforcing foreign key constraints. This is enforced at the schema level and can be managed through SQL commands like ALTER TABLE and CREATE TABLE.

#### Triage

The question asks for an explanation of referential integrity (RI) and how it is enforced via the schema design. We need to explain what RI means, why it's important, and provide examples of how it’s implemented using foreign keys in a database schema.

#### First-Principles Solution
What is Referential Integrity?

Referential integrity ensures that relationships between tables are maintained. It prevents scenarios where data from one table (the "child" or referencing table) doesn’t have a corresponding entry in another table (the "parent" or referenced table). For example, if you have an orders table and a customers table, each order should reference a valid customer ID.

Why is Referential Integrity Important?
Data Consistency: Ensures that the data remains consistent and reliable.
Business Logic Compliance: Helps maintain business rules by preventing invalid or inconsistent records.
Saves Time and Effort: Prevents orphaned records, which can be difficult to manage later.
How is Referential Integrity Enforced?

Referential integrity is enforced through foreign key constraints in the database schema. Here’s a step-by-step explanation:

Creating Foreign Keys:

SQL
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);

In this example, customer_id in the orders table is a foreign key that references customer_id in the customers table. This ensures that every value in orders.customer_id must exist in customers.customer_id.

Enforcing Constraints with ON DELETE and ON UPDATE:

SQL
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
        ON DELETE CASCADE
        ON UPDATE CASCADE
);
ON DELETE CASCADE: If a customer is deleted, all corresponding orders are also deleted.
ON UPDATE CASCADE: If the customer_id in the customers table changes, it will automatically update in the orders table.

Using Indexes: Foreign keys benefit from indexes for faster lookups and to maintain performance during large transactions.

#### Example Scenario

Let’s consider a simple e-commerce application with two tables: products and product_orders.

SQL
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    name VARCHAR(255),
    price DECIMAL(10, 2)
);

CREATE TABLE product_orders (
    order_id INT PRIMARY KEY,
    product_id INT,
    quantity INT,
    FOREIGN KEY (product_id) REFERENCES products(product_id)
);

In this setup:

products.product_id is the primary key.
product_orders.product_id references products.product_id.
If a product is deleted, all related orders will be automatically removed due to the cascading delete rule.

#### Common Mistakes
and Follow-Ups:
Ignoring Foreign Keys: Some developers might overlook foreign keys in favor of simpler logic checks at application level.
Incorrect Constraint Definitions: Misconfigurations like not using ON DELETE CASCADE can lead to orphaned records.

#### Performance Considerations
While indexes improve performance, they also use additional storage and may slow down insertions.

#### Closing Summary

Referential integrity is crucial for maintaining data consistency in a database. It’s enforced via foreign keys and constraints that ensure related records in different tables are always valid. By correctly implementing these rules at the schema level, we can prevent issues like orphaned records and ensure our application behaves as expected.

