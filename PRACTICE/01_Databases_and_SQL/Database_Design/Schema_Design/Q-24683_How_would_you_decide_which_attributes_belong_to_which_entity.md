---
type: "practice"
difficulty: "Intermediate"
---

# How would you decide which attributes belong to which entity while designing a schema?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Surrogate vs Natural Keys for Lookups](<../../../../01_Databases_and_SQL/Database_Design/Schema_Design/Surrogate_vs_Natural_Keys_for_Lookups.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a database schema, deciding which attributes belong to which entity is crucial for ensuring data integrity and optimizing performance. At an intermediate level, this task involves understanding the relationships between entities and identifying primary keys, foreign keys, and other relevant attributes. The key idea here is to model the real-world entities accurately while maintaining normalization principles.

The question essentially asks how you would determine the appropriate attributes for each entity in a given scenario. This process requires a deep understanding of the domain, the data relationships, and the normalization rules. Let's break down the solution step by step:

Understand the Domain: Start by thoroughly understanding the problem domain. For example, if designing a schema for an e-commerce platform, identify the key entities such as customers, products, orders, etc.
Identify Entities and Relationships: Determine which entities are involved in the system and how they relate to each other. Common relationships include one-to-one, one-to-many, many-to-many, and many-to-many through a join table.
Primary Key Identification: Each entity needs a unique identifier (primary key). This can be an auto-incrementing integer or a combination of columns that uniquely identify the record.
Attribute Selection: Attributes should describe the specific characteristics of each entity. Ensure they are relevant and non-redundant by following normalization principles.

Let's consider an example to illustrate these steps:

#### Example
E-commerce Schema

Entities Identification

Customers
Products
Orders
OrderDetails

Primary Keys

Customer can have a primary key like customer_id.
Product might use product_id.
Order could have an order_id.
OrderDetail should also have a composite primary key such as (order_id, product_id).

Attribute Selection

For Customers: customer_id, first_name, last_name, email, phone_number, etc.
For Products: product_id, name, price, description, category_id, etc.
For Orders: order_id, customer_id, order_date, total_amount, etc.
For OrderDetails: (order_id, product_id), quantity, unit_price, etc.

Relationships

A customer can have many orders (Customer to Order is one-to-many).
An order has details about multiple products (Order to OrderDetail is one-to-many via a join table).

#### Triaging the Question

The interviewer might be probing your understanding of:

The importance of domain knowledge.
How to identify and define entities.
Techniques for normalizing data (e.g., 1NF, 2NF, 3NF).
Best practices in schema design.

#### First-Principles Solution

To ensure a robust database schema:

Normalization: Reduce data redundancy by ensuring each piece of non-key information depends on the whole primary key.
Entity Relationships: Ensure that relationships between entities are well-defined and support the business logic.
Data Integrity: Use constraints (e.g., foreign keys, unique indexes) to enforce business rules.

#### Closing Summary

Deciding which attributes belong to which entity is a foundational task in database design. It requires careful consideration of domain knowledge, data relationships, normalization principles, and best practices in schema construction.

#### Common Mistakes/Follow-Ups
Ignoring Domain Knowledge: Failing to fully understand the business processes can lead to poor schema design.
Over-Normalization: Reducing redundancy too much can lead to complex queries and performance issues.
Under-Normalization: Allowing data redundancy increases storage requirements and maintenance costs.

**Code:**

```sql
CREATE TABLE Customers (
    customer_id INT PRIMARY KEY AUTO_INCREMENT,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    email VARCHAR(100) UNIQUE,
    phone_number VARCHAR(20)
);

CREATE TABLE Products (
    product_id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100),
    price DECIMAL(10, 2),
    description TEXT,
    category_id INT,
    FOREIGN KEY (category_id) REFERENCES Categories(category_id)
);

CREATE TABLE Orders (
    order_id INT PRIMARY KEY AUTO_INCREMENT,
    customer_id INT,
    order_date DATETIME,
    total_amount DECIMAL(10, 2),
    FOREIGN KEY (customer_id) REFERENCES Customers(customer_id)
);

CREATE TABLE OrderDetails (
    order_id INT,
    product_id INT,
    quantity INT,
    unit_price DECIMAL(10, 2),
    PRIMARY KEY (order_id, product_id),
    FOREIGN KEY (order_id) REFERENCES Orders(order_id),
    FOREIGN KEY (product_id) REFERENCES Products(product_id)
);
```

