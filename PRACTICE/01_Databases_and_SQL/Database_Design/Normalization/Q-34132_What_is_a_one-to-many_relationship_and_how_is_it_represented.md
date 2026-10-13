---
type: "practice"
difficulty: "Basic"
---

# What is a one-to-many relationship and how is it represented in a normalized schema?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Redundancy and Anomalies](<../../../../01_Databases_and_SQL/Database_Design/Normalization/Redundancy_and_Anomalies.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In database design, understanding relationships between tables is crucial to ensure data integrity and efficiency. A one-to-many (1:N) relationship describes a situation where one record in Table A can be associated with multiple records in Table B, but each record in Table B is only associated with one record in Table A. In the context of normalization, this type of relationship needs careful handling to maintain data consistency and reduce redundancy.

#### Triaging the Question

The question asks us to explain what a one-to-many (1:N) relationship is and how it's represented in a normalized schema. The interviewer is likely probing your understanding of relational database concepts, particularly normalization techniques that handle such relationships effectively.

#### First-Principles Solution
What Is a One-to-Many Relationship?

A one-to-many relationship means that each record in the "one" table can be linked to multiple records in the "many" table. For example, consider an online store where a customer can place many orders but each order is placed by only one customer. Here, the Customers table (the “one” side) and the Orders table (the “many” side) have this relationship.

Representing One-to-Many Relationships in Normalized Schema

In normalization, we typically represent a one-to-many relationship using foreign keys to maintain referential integrity. Let’s assume you are designing a database for an online store with two tables: Customers and Orders.

Table: Customers

Column Name	Data Type
customer_id (PK)	INT
first_name	VARCHAR(50)
last_name	VARCHAR(50)
email	VARCHAR(100)

Table: Orders

Column Name	Data Type
order_id (PK)	INT
customer_id (FK)	INT
order_date	DATE
total_amount	DECIMAL

In this schema, the customer_id column in the Orders table is a foreign key that references the customer_id column in the Customers table. This setup ensures that each order is linked to exactly one customer and allows multiple orders per customer.

Why Normalize?

Normalization helps avoid data redundancy and inconsistencies by ensuring that each piece of information appears only once in the database. It also simplifies the management of data, making it easier to maintain and query.

#### Common Mistakes
Forgetting Foreign Keys: Not including foreign keys can lead to orphan records or inconsistent data.
Over-Normalization: Splitting tables too much can make queries complex and reduce performance.
Not Considering Performance Trade-offs: While normalization is essential, overly normalized schemas might slow down read operations.

#### Closing Summary

One-to-many relationships in databases are critical for representing hierarchical associations between entities like customers and orders. By using foreign keys, we ensure referential integrity and data consistency. Proper normalization techniques help manage these relationships effectively, though it’s important to balance this with performance considerations.

**Comparison:**

| Customers | customer_id, first_name, last_name, email |
| Orders    | order_id, customer_id, order_date, total_amount |

**Code:**

```sql
-- Table: Customers
CREATE TABLE Customers (
    customer_id INT PRIMARY KEY,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    email VARCHAR(100)
);

-- Table: Orders
CREATE TABLE Orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    order_date DATE,
    total_amount DECIMAL,
    FOREIGN KEY (customer_id) REFERENCES Customers(customer_id)
);
```

