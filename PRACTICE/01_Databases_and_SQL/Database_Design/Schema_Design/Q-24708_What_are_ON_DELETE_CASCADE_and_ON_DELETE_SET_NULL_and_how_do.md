---
type: "practice"
difficulty: "Deep"
---

# What are ON DELETE CASCADE and ON DELETE SET NULL and how do they relate to ER relationships?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Polymorphic Associations](<../../../../01_Databases_and_SQL/Database_Design/Schema_Design/Polymorphic_Associations.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

ON DELETE CASCADE and ON DELETE SET NULL are important concepts in database design, particularly when dealing with referential integrity between tables. They define what happens to related records in one table when a record is deleted from another table. Understanding these operations can help ensure data consistency and prevent orphaned or inconsistent records.

To address the question comprehensively, let's break down each concept step by step:

#### Triaging the Question

What the Question is Really Asking:

The question asks for an explanation of ON DELETE CASCADE and ON DELETE SET NULL, and their relationship to Entity Relationship (ER) diagrams.
The interviewer is probing your understanding of referential integrity, database normalization, and how different constraints can be applied.
Key Concepts

#### Assumptions

We are working with a relational database management system (RDBMS) like MySQL or PostgreSQL.
We have two tables where one table has a foreign key referencing the primary key of another table. This is a common scenario in ER diagrams, where entities and their relationships are represented.
Step-by-Step Solution
ON DELETE CASCADE

ON DELETE CASCADE is an action that automatically deletes related records from child tables when a record is deleted from the parent table. It ensures referential integrity by cascading the delete operation down to all dependent rows in the child tables.

#### Example
Suppose we have two tables, Order and Customer. The Order table has a foreign key column customer_id referencing the primary key of the Customer table.

SQL
CREATE TABLE Customer (
    customer_id INT PRIMARY KEY,
    name VARCHAR(255)
);

CREATE TABLE Order (
    order_id INT PRIMARY KEY,
    customer_id INT,
    FOREIGN KEY (customer_id) REFERENCES Customer(customer_id) ON DELETE CASCADE
);

If a Customer is deleted, all associated orders will be automatically removed from the Order table.

ON DELETE SET NULL

ON DELETE SET NULL is another action that sets the foreign key column to null when a record in the parent table is deleted. This approach avoids orphaned records but may require additional logic to handle nulls properly.

#### Example
Using the same tables as before:

SQL
CREATE TABLE Order (
    order_id INT PRIMARY KEY,
    customer_id INT,
    FOREIGN KEY (customer_id) REFERENCES Customer(customer_id) ON DELETE SET NULL
);

If a Customer is deleted, all orders associated with that customer will have their customer_id set to null in the Order table.

#### Common Mistakes
and Follow-Ups

#### Common Mistakes

Forgetting to specify the foreign key constraint when creating tables.
Misunderstanding when to use CASCADE versus SET NULL.
Not considering performance implications, as cascading deletes can be costly on large databases.

#### Follow-Up
Questions:

How would you handle a situation where both ON DELETE CASCADE and ON DELETE SET NULL are not suitable?
In what scenarios might you choose one over the other?

#### Closing Summary

In summary, ON DELETE CASCADE and ON DELETE SET NULL are crucial for maintaining referential integrity in relational databases. While CASCADE automatically deletes related records to avoid orphaned data, SET NULL sets foreign keys to null. The choice between these depends on your specific use case and the desired level of data consistency.

Self-Check

Correctness + completeness confirmed.

**Code:**

```sql
-- Example of ON DELETE CASCADE
CREATE TABLE Customer (
    customer_id INT PRIMARY KEY,
    name VARCHAR(255)
);

CREATE TABLE Order (
    order_id INT PRIMARY KEY,
    customer_id INT,
    FOREIGN KEY (customer_id) REFERENCES Customer(customer_id) ON DELETE CASCADE
);
```

