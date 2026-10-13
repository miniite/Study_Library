---
type: "practice"
difficulty: "Basic"
---

# What are the common data types in PostgreSQL and when would you use SERIAL vs INTEGER?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ UUID vs Serial Keys](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/UUID_vs_Serial_Keys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with databases, understanding data types is crucial as they dictate how the data will be stored and processed. In PostgreSQL, a popular SQL database system, there are several common data types that you can use based on your needs. Today, we'll focus on two specific integer types: INTEGER and SERIAL. We'll explore when to use each one by diving into their core characteristics.

Let's start with a brief summary: Both INTEGER and SERIAL are used for storing numeric values in PostgreSQL, but they serve slightly different purposes. An INTEGER is just a regular integer that you manually assign; whereas, a SERIAL type automatically generates an incrementing value each time a new record is inserted.

**Comparison:**

```
Data Type	Description
INTEGER	Regular integer that requires manual assignment.
SERIAL	An auto-incrementing integer, which is used as a surrogate key for primary keys.

N/A

SQL
Copy
-- Inserting data into the tables

INSERT INTO employees (id, name) VALUES (1, 'John Doe');
INSERT INTO orders (customer_name, product_name) VALUES ('Alice Smith', 'Laptop');
```

**Code:**

```sql
-- Example usage of INTEGER and SERIAL

-- Define a table with an INTEGER column
CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR(100)
);

-- Define a table with a SERIAL column
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    customer_name VARCHAR(100),
    product_name VARCHAR(100)
);
Data Type	Description
INTEGER	Regular integer that requires manual assignment.
SERIAL	An auto-incrementing integer, which is used as a surrogate key for primary keys.
```

