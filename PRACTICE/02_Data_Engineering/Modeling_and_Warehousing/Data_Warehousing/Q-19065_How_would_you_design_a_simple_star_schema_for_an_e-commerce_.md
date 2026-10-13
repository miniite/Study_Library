---
type: "practice"
difficulty: "Intermediate"
---

# How would you design a simple star schema for an e-commerce orders reporting use case?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Distribution Keys](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Data_Warehousing/Distribution_Keys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we need to design a simple star schema for an e-commerce orders reporting use case. A star schema is a data warehousing model where fact tables are surrounded by dimension tables—forming a shape resembling a star when visualized. The key idea here is to ensure that the schema supports efficient and easy querying of order-related metrics while maintaining data integrity.

The question is probing your understanding of how to structure relational databases for reporting purposes, particularly in an e-commerce context. We will assume the following:

Orders can be linked through a common order ID.
Each order has multiple line items (products).
There are various types of dimensions such as customer demographics, time period, and product categories.

Let's break down the schema design step by step from first principles:

Fact Table - Orders: This table will store all the transactional data related to orders.
Dimension Tables:
Customers: Stores information about customers (ID, name, address, etc.)
Products: Contains details of each product sold in the e-commerce platform (ID, name, category, price, etc.)
Time: Captures temporal data such as order date and time.
Orders_History (optional): A separate table to record changes or historical data for orders.

We will now walk through the design process with a focus on creating an efficient schema that supports various reporting needs.

Fact Table: Orders

SQL
CREATE TABLE Orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    product_id INT,
    quantity INT,
    price DECIMAL(10, 2),
    order_date DATE,
    status VARCHAR(50)
);

Dimension Table: Customers

SQL
CREATE TABLE Customers (
    customer_id INT PRIMARY KEY,
    first_name VARCHAR(100),
    last_name VARCHAR(100),
    email VARCHAR(100),
    address VARCHAR(256),
    city VARCHAR(100),
    state VARCHAR(100),
    country VARCHAR(100),
    zip_code VARCHAR(20)
);

Dimension Table: Products

SQL
CREATE TABLE Products (
    product_id INT PRIMARY KEY,
    name VARCHAR(100),
    category VARCHAR(50),
    price DECIMAL(10, 2),
    supplier VARCHAR(100),
    stock_quantity INT
);

Dimension Table: Time

SQL
CREATE TABLE Time (
    date_id INT PRIMARY KEY,
    order_date DATE,
    year INT,
    quarter INT,
    month INT,
    week INT,
    day INT
);

Optional Dimension Table: Orders_History (if needed)

SQL
CREATE TABLE Orders_History (
    history_id INT PRIMARY KEY,
    order_id INT,
    field_name VARCHAR(100),
    old_value VARCHAR(256),
    new_value VARCHAR(256),
    change_date TIMESTAMP,
    user_id INT
);

Next, let’s consider some common mistakes and follow-up questions:

Mistake: Not normalizing the data. For instance, customer_id should be a foreign key in the fact table Orders, linking back to Customers.

#### Follow-Up
How do you handle duplicate customers or products? Do you merge them into one entry?
Mistake: Failing to include essential dimensions like time. Time is crucial for trend analysis.

#### Follow-Up
What are some temporal aggregation strategies (e.g., monthly, quarterly)?

**Comparison:**

```
Order ID	Customer ID	Product ID	Quantity	Price	Order Date	Status
1001	202	3001	5	9.99	2023-06-01	Shipped
```

