---
type: "practice"
difficulty: "Intermediate"
---

# What are the storage trade-offs between a star schema and a snowflake schema?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Star Schemas and Dimension Keys](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Star_Schemas_and_Dimension_Keys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In data warehousing, the choice between a star schema and a snowflake schema depends significantly on the specific requirements of your data analysis and storage needs. A star schema organizes data around fact tables with dimension tables that are denormalized, creating a simpler structure but potentially more redundant data. On the other hand, a snowflake schema normalizes dimension tables, offering better data integrity at the cost of increased complexity.

To understand the trade-offs between these schemas, we need to consider storage efficiency, query performance, and the potential for joins. Let's break this down step by step from first principles.

**Comparison:**

```
Fact Table
Star Schema
Denormalized Dimensions
Snowflake Schema
Normalized Products Dim
Categories
Suppliers
Normalized Stores Dim
Normalized Times Dim
```

**Code:**

```sql
-- Example fact table in a star schema
CREATE TABLE sales_fact (
    sale_id INT PRIMARY KEY,
    product_key INT NOT NULL,
    store_key INT NOT NULL,
    time_key INT NOT NULL,
    quantity_sold INT,
    price DECIMAL(10, 2)
);

-- Corresponding dimension tables
CREATE TABLE products_dim (
    product_key INT PRIMARY KEY,
    product_name VARCHAR(100),
    category_id INT,
    supplier_id INT
);
CREATE TABLE stores_dim (
    store_key INT PRIMARY KEY,
    store_name VARCHAR(150),
    city_id INT
);
CREATE TABLE times_dim (
    time_key INT PRIMARY KEY,
    date DATE,
    month INT,
    year INT
);

-- Example fact table in a snowflake schema
CREATE TABLE sales_fact (
    sale_id INT PRIMARY KEY,
    product_id INT NOT NULL REFERENCES products_dim(product_id),
    store_id INT NOT NULL REFERENCES stores_dim(store_id),
    time_id INT NOT NULL REFERENCES times_dim(time_id),
    quantity_sold INT,
    price DECIMAL(10, 2)
);

-- Corresponding dimension tables
CREATE TABLE products_dim (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100),
    category_id INT NOT NULL REFERENCES categories_dim(category_id),
    supplier_id INT NOT NULL REFERENCES suppliers_dim(supplier_id)
);
CREATE TABLE stores_dim (
    store_id INT PRIMARY KEY,
    store_name VARCHAR(150),
    city_id INT
);
CREATE TABLE times_dim (
    time_id INT PRIMARY KEY,
    date DATE,
    month INT,
    year INT
);
```

**Example input / output:**

```text
Input for Star Schema:
SELECT
    s.quantity_sold,
    p.product_name,
    t.date,
    st.store_name
FROM sales_fact s
JOIN products_dim p ON s.product_key = p.product_key
JOIN stores_dim st ON s.store_key = st.store_key
JOIN times_dim t ON s.time_key = t.time_key;

Output for Star Schema:
+--------------+-----------------+------------+---------------+
| quantity_sold | product_name    | date       | store_name    |
+--------------+-----------------+------------+---------------+
| 10           | Laptop          | 2023-06-01 | Tech Mart     |
| 20           | Smartphone      | 2023-06-15 | Mobile Hub    |
| 30           | Tablet          | 2023-06-30 | Gadget Plaza  |
+--------------+-----------------+------------+---------------+

Input for Snowflake Schema:
SELECT
    s.quantity_sold,
    p.product_name,
    t.date,
    st.store_name
FROM sales_fact s
JOIN products_dim p ON s.product_id = p.product_id
JOIN stores_dim st ON s.store_key = st.store_key
JOIN times_dim t ON s.time_key = t.time_key;

Output for Snowflake Schema:
+--------------+-----------------+------------+---------------+
| quantity_sold | product_name    | date       | store_name    |
+--------------+-----------------+------------+---------------+
| 10           | Laptop          | 2023-06-01 | Tech Mart     |
| 20           | Smartphone      | 2023-06-15 | Mobile Hub    |
| 30           | Tablet          | 2023-06-30 | Gadget Plaza  |
+--------------+-----------------+------------+---------------+
```

**Data:**

```json
{
  "sales_fact": {
    "sale_id": [1, 2, 3],
    "product_key": [101, 102, 103],
    "store_key": [501, 502, 503],
    "time_key": [701, 702, 703],
    "quantity_sold": [10, 20, 30],
    "price": [19.99, 49.99, 69.99]
  },
  "products_dim": {
    "product_key": [101, 102, 103],
    "product_name": ["Laptop", "Smartphone", "Tablet"],
    "category_id": [1, 2, 3],
    "supplier_id": [4, 5, 6]
  },
  "stores_dim": {
    "store_key": [501, 502, 503],
    "store_name": ["Tech Mart", "Mobile Hub", "Gadget Plaza"],
    "city_id": [1001, 1002, 1003]
  },
  "times_dim": {
    "time_key": [701, 702, 703],
    "date": ["2023-06-01", "2023-06-15", "2023-06-30"],
    "month": [6, 6, 6],
    "year": [2023, 2023, 2023]
  }
}
```

**Flow:**

1. @startuml
2. title Star vs Snowflake Schema
3. actor User
4. database "Star Schema" as SS
5. database "Snowflake Schema" as SF
6. User->SS: Query for Sale Summary
7. SS->SS: JOIN Fact with Dimensions
8. SS->User: Return Summary Data
9. User->SF: Query for Sale Summary
10. SF->SF: JOIN Fact with Normalized Dim Tables
11. SF->User: Return Summary Data
12. @enduml

