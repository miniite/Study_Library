---
type: "practice"
difficulty: "Basic"
---

# What is a star schema and how does it differ from a snowflake schema?

**Difficulty:** Basic · **Levels:** Fresher · [↑ Relationship Modeling](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Relationship_Modeling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A star schema is a database design in which a central fact table is surrounded by multiple dimension tables, forming a "star" shape. It’s designed to optimize read performance and simplify query execution, particularly useful in data warehousing environments.

To understand the differences between a star schema and a snowflake schema, let's first define each:

Star Schema:

A central fact table with multiple dimension tables directly linked to it.
Dimensions can be either atomic (simple) or composite but are usually not normalized.
Simplifies query execution as all relevant data is stored in one place.

Snowflake Schema:

An extension of the star schema where dimensions are further broken down into multiple levels, creating a "snowflaking" effect.
Dimensions can be normalized and split into smaller tables to reduce redundancy but may complicate queries.

The key difference lies in how the dimension tables are structured. While both schemas aim for efficiency, they handle data normalization differently, impacting query complexity and storage requirements.

**Code:**

```

-- Example of a star schema: CREATE TABLE orders ( order_id INT PRIMARY KEY, product_id INT, customer_id INT, order_date DATE );

CREATE TABLE products ( product_id INT PRIMARY KEY, product_name VARCHAR(100) );

CREATE TABLE customers ( customer_id INT PRIMARY KEY, customer_name VARCHAR(100), address VARCHAR(255) );

CODE
Copy

```sql
-- Example of a snowflake schema:
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    product_id INT,
    customer_id INT,
    order_date DATE
);

CREATE TABLE products (
    product_id INT PRIMARY KEY,
    product_category_id INT,
    product_name VARCHAR(100)
);

CREATE TABLE product_categories (
    category_id INT PRIMARY KEY,
    category_name VARCHAR(50),
    parent_category_id INT
);
```

