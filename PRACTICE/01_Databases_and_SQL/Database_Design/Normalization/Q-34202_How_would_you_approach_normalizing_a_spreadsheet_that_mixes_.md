---
type: "practice"
difficulty: "Intermediate"
---

# How would you approach normalizing a spreadsheet that mixes customer order and product details in one sheet?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Redundancy and Anomalies](<../../../../01_Databases_and_SQL/Database_Design/Normalization/Redundancy_and_Anomalies.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

When faced with a mixed spreadsheet containing both customer orders and product details, the key is to normalize it by separating the data into multiple tables based on functional dependencies, ensuring that each table has only one reason to change. This involves identifying entities (customers, products), defining their attributes, and creating relationships between them.

#### Triaging the Question

The core of this question revolves around normalization—the process of organizing data in a database to reduce redundancy and improve data integrity. Specifically, we need to address a mixed spreadsheet that contains both order details and product information in one sheet. The interviewer is probing your understanding of normalization concepts and your ability to design an efficient data model.

#### First-Principles Solution
Identify Entities: Determine the main entities involved—customers and products.
Define Attributes: List attributes for each entity:
Customers: customer_id, name, email, phone
Products: product_id, name, description, price
Create Relationships:
Orders involve both customers and products, so they can be a separate entity.
Normalization Steps:
Create a table for each identified entity (customers, products).
Introduce an order table that links to the customer and product tables.
Step-by-Step Approach
Identify Entities:
Customers: Store details like name, email, and phone.
Products: Store details about the product such as name, description, and price.
Define Attributes:
Customer Table: customer_id, name, email, phone
Product Table: product_id, name, description, price
Order Relationship:
Order Table: order_id, customer_id (foreign key), product_id (foreign key), quantity, order_date

#### Example
Schema
SQL
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    name VARCHAR(255),
    email VARCHAR(255),
    phone VARCHAR(15)
);

CREATE TABLE products (
    product_id INT PRIMARY KEY,
    name VARCHAR(255),
    description TEXT,
    price DECIMAL(10, 2)
);

CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    product_id INT,
    quantity INT,
    order_date DATE,
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id),
    FOREIGN KEY (product_id) REFERENCES products(product_id)
);

#### Closing Summary
and Common Mistakes

By normalizing the spreadsheet, we ensure that data is structured in a way that minimizes redundancy and improves data integrity. This approach helps maintain consistency and makes it easier to manage updates and queries.

#### Common Mistakes

Failing to identify all entities correctly.
Over-normalization leading to excessive join operations during queries.
Not defining foreign keys properly, which can lead to referential integrity issues.

#### Example
Worked Input -> Output

Given the input data in a mixed format:

order_id	customer_name	product_name	quantity
1	John Doe	Widget A	2
2	Jane Smith	Gadget B	3

After normalization, the transformed data would be:

Customers Table:
customer_id=1, name="John Doe", email="", phone=""
Products Table:
product_id=1, name="Widget A", description="", price=""
Orders Table:
order_id=1, customer_id=1, product_id=1, quantity=2, order_date="YYYY-MM-DD"
Self-Check

This solution is correct and complete.

**Comparison:**

| Customers Table                | Products Table             | Orders Table            |
|---------------------------------|----------------------------|-------------------------|
| customer_id (PK)                | product_id (PK)            | order_id (PK)           |
| name                            | name                       | customer_id (FK)        |
| email                           | description                | product_id (FK)         |
| phone                           | price                      | quantity                |

**Code:**

```sql
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    name VARCHAR(255),
    email VARCHAR(255),
    phone VARCHAR(15)
);

CREATE TABLE products (
    product_id INT PRIMARY KEY,
    name VARCHAR(255),
    description TEXT,
    price DECIMAL(10, 2)
);

CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    product_id INT,
    quantity INT,
    order_date DATE,
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id),
    FOREIGN KEY (product_id) REFERENCES products(product_id)
);
```

**Example input / output:**

```text
SELECT o.order_id, c.name AS customer_name, p.name AS product_name, o.quantity
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
JOIN products p ON o.product_id = p.product_id;
-- Output:
| order_id | customer_name    | product_name  | quantity |
|----------|------------------|---------------|----------|
| 1        | John Doe         | Widget A      | 2        |
| 2        | Jane Smith       | Gadget B      | 3        |
```

**Data:**

```sql
INSERT INTO customers VALUES
(1, 'John Doe', '', ''),
(2, 'Jane Smith', '', '');

INSERT INTO products VALUES
(1, 'Widget A', '', 50.00),
(2, 'Gadget B', '', 75.00);

INSERT INTO orders VALUES
(1, 1, 1, 2, '2023-10-01'),
(2, 2, 2, 3, '2023-10-01');
```

**Flow:**

1. one-to-many
2. Customers
3. Orders
4. Products

