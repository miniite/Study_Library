---
type: "practice"
difficulty: "Intermediate"
---

# What is a trigger and give a scenario where you would use one.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ RETURNING Clause](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/RETURNING_Clause.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In PostgreSQL, a trigger is a special function that gets automatically executed in response to certain events on a specific table. Triggers are often used to maintain data integrity, enforce business rules, or update related tables when an event occurs.

#### Triaging the Question

The question asks us to explain what a trigger is and provide a scenario where it would be useful. The core of this question is understanding how triggers work in PostgreSQL and appreciating their importance for database management.

#### Assumptions

We assume basic knowledge of SQL and relational databases.
We will focus on the use case of maintaining data integrity and ensuring consistent state changes across related tables.

#### First Principles Solution

Triggers are functions that execute automatically when a specific event occurs on a table, such as an INSERT, UPDATE, or DELETE. These functions can be written in PL/pgSQL (PostgreSQL's procedural language) and typically perform operations like logging changes, enforcing constraints, or updating related tables.

#### Example Scenario
Maintaining Consistency Across Related Tables

#### Cons
ider a scenario where we have two tables: orders and order_items. Each order can contain multiple items. When an item is added to an order (i.e., an entry is inserted into the order_items table), we want to ensure that the total quantity of items for each product in the products table is updated accordingly.

Orders Table:

SQL
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    customer_id INT NOT NULL,
    total_amount DECIMAL(10, 2) NOT NULL
);

Order Items Table:

SQL
CREATE TABLE order_items (
    id SERIAL PRIMARY KEY,
    order_id INT NOT NULL REFERENCES orders(id),
    product_id INT NOT NULL REFERENCES products(id),
    quantity INT NOT NULL
);

Products Table:

SQL
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    stock_quantity INT NOT NULL
);

To maintain the stock_quantity in the products table, we can use a trigger on the order_items table.

Trigger Function:

SQL
CREATE OR REPLACE FUNCTION update_product_stock() RETURNS TRIGGER AS $$
BEGIN
    IF TG_OP = 'INSERT' THEN
        UPDATE products SET stock_quantity = stock_quantity - NEW.quantity WHERE id = NEW.product_id;
    ELSIF TG_OP = 'DELETE' THEN
        UPDATE products SET stock_quantity = stock_quantity + OLD.quantity WHERE id = OLD.product_id;
    END IF;
    RETURN NULL;
END;
$$ LANGUAGE plpgsql;

Creating the Trigger:

SQL
CREATE TRIGGER update_product_stock_trigger
AFTER INSERT OR DELETE ON order_items
FOR EACH ROW EXECUTE FUNCTION update_product_stock();

#### Closing Summary
and Common Mistakes

Triggers are a powerful feature in PostgreSQL that allow you to enforce complex business rules or maintain data integrity without writing repetitive code. The example scenario demonstrates how triggers can be used to keep related tables consistent.

#### Common Mistakes

Forgetting to account for edge cases, such as what happens when an item is updated rather than inserted or deleted.
Not considering the performance implications of frequently executing functions on large datasets.
Overusing triggers instead of leveraging SQL constraints and other built-in mechanisms where possible.

**Comparison:**

```
has
ORDERS
ORDER_ITEMS
orders
int
id
int
customer_id
decimal
total_amount
order_items
int
id
int
order_id
int
product_id
int
quantity
products
int
id
varchar(100)
name
int
stock_quantity
```

**Code:**

```sql
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    customer_id INT NOT NULL,
    total_amount DECIMAL(10, 2) NOT NULL
);

CREATE TABLE order_items (
    id SERIAL PRIMARY KEY,
    order_id INT NOT NULL REFERENCES orders(id),
    product_id INT NOT NULL REFERENCES products(id),
    quantity INT NOT NULL
);

CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    stock_quantity INT NOT NULL
);

CREATE OR REPLACE FUNCTION update_product_stock() RETURNS TRIGGER AS $$
BEGIN
    IF TG_OP = 'INSERT' THEN
        UPDATE products SET stock_quantity = stock_quantity - NEW.quantity WHERE id = NEW.product_id;
    ELSIF TG_OP = 'DELETE' THEN
        UPDATE products SET stock_quantity = stock_quantity + OLD.quantity WHERE id = OLD.product_id;
    END IF;
    RETURN NULL;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER update_product_stock_trigger
AFTER INSERT OR DELETE ON order_items
FOR EACH ROW EXECUTE FUNCTION update_product_stock();
```

