---
type: "practice"
difficulty: "Basic"
---

# How would you briefly describe the Kimball approach to someone new to warehousing?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Kimball Approach](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Kimball_Approach.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

The Kimball approach in data warehousing is a method that prioritizes data ease of use and flexibility over strict normalization, making it highly suitable for business intelligence and analytics. At its core, this approach focuses on creating star or snowflake schemas to organize your data warehouse efficiently.

To explain the Kimball approach to someone new to warehousing:

#### Overview
The Kimball approach emphasizes building a data warehouse that is optimized for querying and reporting rather than transactional operations.
Key Components: It uses star or snowflake schemas, which are designed to be simple and easy to understand while still providing the necessary flexibility for complex queries.
Star Schema: This schema consists of fact tables (which store detailed data) surrounded by dimension tables (which provide context). The central hub is the fact table, with each spoke representing a dimension.
Snowflake Schema: An extension of the star schema where some dimension tables are further decomposed into smaller, related tables to reduce redundancy and improve manageability.

The Kimball approach also advocates for a gradual build strategy, allowing businesses to start small and scale up as needed. This ensures that the initial implementation is manageable and can evolve over time based on changing requirements.

**Comparison:**

| Fact Table (Sales) | Dimension Tables          |
|--------------------|---------------------------|
| sale_id, product_key, customer_key, store_key, sale_date, quantity_sold, revenue | products_dim (product_name, category_id, supplier_id), customers_dim (customer_name, city), stores_dim (store_name, location_city) |

**Code:**

```sql
-- Example of a star schema fact table
CREATE TABLE sales_fact (
    sale_id INT PRIMARY KEY,
    product_key INT,
    customer_key INT,
    store_key INT,
    sale_date DATE,
    quantity_sold INT,
    revenue DECIMAL(10, 2)
);

-- Example dimension tables
CREATE TABLE products_dim (
    product_key INT PRIMARY KEY,
    product_name VARCHAR(50),
    category_id INT,
    supplier_id INT
);

CREATE TABLE customers_dim (
    customer_key INT PRIMARY KEY,
    customer_name VARCHAR(100),
    city VARCHAR(50)
);

CREATE TABLE stores_dim (
    store_key INT PRIMARY KEY,
    store_name VARCHAR(100),
    location_city VARCHAR(50)
);
```

**Example input / output:**

```text
-- Example query to retrieve sales data by product and location
SELECT
    p.product_name,
    c.customer_name,
    s.store_name,
    f.sale_date,
    f.quantity_sold,
    f.revenue
FROM
    sales_fact f
JOIN
    products_dim p ON f.product_key = p.product_key
JOIN
    customers_dim c ON f.customer_key = c.customer_key
JOIN
    stores_dim s ON f.store_key = s.store_key;
```

**Data:**

```json
{
  "sales_fact": {
    "sale_id": [1, 2, 3],
    "product_key": [101, 102, 103],
    "customer_key": [201, 202, 203],
    "store_key": [301, 302, 303],
    "sale_date": ["2023-01-01", "2023-02-01", "2023-03-01"],
    "quantity_sold": [5, 7, 9],
    "revenue": [15.5, 21.8, 27.3]
  },
  "products_dim": {
    "product_key": [101, 102, 103],
    "product_name": ["Phone", "Laptop", "Tablet"],
    "category_id": [1, 2, 3],
    "supplier_id": [4, 5, 6]
  },
  "customers_dim": {
    "customer_key": [201, 202, 203],
    "customer_name": ["Alice", "Bob", "Charlie"],
    "city": ["New York", "Los Angeles", "Chicago"]
  },
  "stores_dim": {
    "store_key": [301, 302, 303],
    "store_name": ["Store A", "Store B", "Store C"],
    "location_city": ["Boston", "San Francisco", "Seattle"]
  }
}
```

**Flow:**

1. Central hub
2. Spoke 1
3. Spoke 2
4. Spoke 3
5. Fact Table
6. Star Schema
7. Fact Table
8. Dimension Table - Products
9. Dimension Table - Customers
10. Dimension Table - Stores

