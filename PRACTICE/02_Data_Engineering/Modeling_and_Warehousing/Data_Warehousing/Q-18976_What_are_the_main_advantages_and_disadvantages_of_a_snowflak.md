---
type: "practice"
difficulty: "Intermediate"
---

# What are the main advantages and disadvantages of a snowflake schema compared to a star schema?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Distribution Keys](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Data_Warehousing/Distribution_Keys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

The snowflake schema and the star schema are two popular dimensional modeling techniques used in data warehousing. The primary advantage of the snowflake schema is its ability to reduce storage space by breaking down a large fact table into smaller, normalized tables. However, this comes at the cost of increased complexity and potentially slower query performance due to the need for more joins.

#### Triaging the Question

The question asks us to compare the main advantages and disadvantages of using a snowflake schema versus a star schema in data warehousing. The key aspects we will explore are storage efficiency, complexity, and query performance.

#### Assumptions
We assume familiarity with basic concepts of data warehousing and dimensional modeling.
Both schemas are used for multidimensional analysis (OLAP) over fact tables.
We consider a typical business scenario involving large datasets to highlight the differences more clearly.
Step-by-Step Solution from First Principles:
Star Schema
Fact Table: A central, denormalized table containing the primary data points.
Dimension Tables: Small, normalized tables that contain attributes related to the fact table.
Key Characteristics:
Simplicity in design: No joins required for most queries.
Easy to implement and manage.
Snowflake Schema
Fact Table: A central, denormalized table containing the primary data points.
Dimension Tables: Normalized into multiple levels (levels 0, 1, 2).
Key Characteristics:
Higher storage efficiency: Reduces redundancy by breaking down large dimension tables.
Increased normalization: Can lead to more complex schemas with deeper joins.
Advantages of Snowflake Schema:
Storage Efficiency: By normalizing dimension tables into smaller pieces (levels 1, 2), redundant data is reduced, leading to lower storage requirements.
Scalability: Easier to scale and maintain as the schema can be managed more granularly.
Query Performance for Complex Queries: Smaller fact tables might lead to faster query execution times in certain scenarios.
Disadvantages of Snowflake Schema:
Complexity: More complex schemas require deeper joins, which can complicate ETL processes and make data modeling harder.
Query Performance Overhead: The increased number of joins required for queries can slow down performance, especially when dealing with large datasets.
Maintenance Overhead: Managing a snowflake schema involves more effort in terms of schema design, indexing, and query optimization.
Advantages of Star Schema:
Simplicity: Easier to understand and implement due to fewer joins required for most queries.
Performance: Generally faster query performance due to simpler join operations.
Ease of Use: Simpler schema makes it easier to manage and maintain, especially for less experienced data engineers.
Disadvantages of Star Schema:
Storage Inefficiency: Higher redundancy in dimension tables can lead to increased storage costs.
Limited Scalability: Harder to scale as the number of dimensions grows since they are not split into multiple levels.
Query Performance for Complex Queries: For very complex queries involving deep joins, performance might degrade.

#### Example Scenario

#### Cons
ider an e-commerce company with a large dataset tracking product sales over time. The star schema would keep all related data in one fact table and dimension tables like products, orders, and customers. A snowflake schema might split the products dimension into multiple levels such as brand, category, and subcategory.

#### Closing Summary

The choice between a star and a snowflake schema depends on the specific requirements of your data warehousing environment. Star schemas are simpler and more performant for typical queries, but they can become storage inefficient with large datasets. Snowflake schemas offer better storage efficiency and scalability but come at the cost of increased complexity and potential performance overhead.

#### Common Mistakes/Follow-Ups
Ignoring Schema Complexity: Overlooking the added complexity in schema design and maintenance.
Neglecting Query Performance: Failing to consider how query performance will be affected by the chosen schema.
Not Evaluating Storage Needs: Ignoring storage costs, which can be significant with a large number of dimension tables.

**Comparison:**

```
Snowflake Schema
Star Schema
JOIN
JOIN
sales_fact
products_dim_level_0
products_dim_level_1
```

**Code:**

```

-- Example Star Schema Fact Table CREATE TABLE sales_fact ( sale_id INT, product_key INT, order_date DATE, quantity INT, amount DECIMAL(10,2) );

-- Dimension Tables in Star Schema CREATE TABLE products_dim ( product_key INT PRIMARY KEY, brand VARCHAR(50), category VARCHAR(50), subcategory VARCHAR(50) );

CODE
Copy

```sql
-- Example Snowflake Schema Fact Table with Normalized Dimensions
CREATE TABLE sales_fact (
    sale_id INT,
    product_key INT,
    order_date DATE,
    quantity INT,
    amount DECIMAL(10,2)
);

-- Level 0 and Level 1 Dimension Tables in Snowflake Schema
CREATE TABLE products_dim_level_0 (
    product_key INT PRIMARY KEY,
    brand VARCHAR(50)
);

CREATE TABLE products_dim_level_1 (
    product_key INT,
    category VARCHAR(50),
    FOREIGN KEY (product_key) REFERENCES sales_fact(product_key)
);
```

**Example input / output:**

```text
-- Input: Star Schema Query to Join Fact and Dimension Tables
SELECT sf.sale_id, p.brand, p.category, sf.order_date, sf.quantity, sf.amount
FROM sales_fact sf
JOIN products_dim p ON sf.product_key = p.product_key;

-- Output:
sale_id | brand  | category   | order_date     | quantity | amount
--------+--------+-----------+----------------+----------+--------
1       | Brand A| Category X | 2023-10-01     | 5        | 150.00
2       | Brand B| Category Y | 2023-10-02     | 10       | 200.00

-- Input: Snowflake Schema Query to Join Fact and Dimension Tables
SELECT sf.sale_id, p_level_0.brand, p_level_1.category, sf.order_date, sf.quantity, sf.amount
FROM sales_fact sf
JOIN products_dim_level_0 p_level_0 ON sf.product_key = p_level_0.product_key
JOIN products_dim_level_1 p_level_1 ON sf.product_key = p_level_1.product_key;

-- Output:
sale_id | brand  | category   | order_date     | quantity | amount
--------+--------+-----------+----------------+----------+--------
1       | Brand A| Category X | 2023-10-01     | 5        | 150.00
2       | Brand B| Category Y | 2023-10-02     | 10       | 200.00
```

**Data:**

```json
{
  "sales_fact": [
    {"sale_id": 1, "product_key": 234, "order_date": "2023-10-01", "quantity": 5, "amount": 150.00},
    {"sale_id": 2, "product_key": 678, "order_date": "2023-10-02", "quantity": 10, "amount": 200.00}
  ],
  "products_dim_level_0": [
    {"product_key": 234, "brand": "Brand A"},
    {"product_key": 678, "brand": "Brand B"}
  ],
  "products_dim_level_1": [
    {"product_key": 234, "category": "Category X"},
    {"product_key": 678, "category": "Category Y"}
  ]
}
```

**Flow:**

1. High
2. Simple
3. Faster
4. Low
5. Complex
6. Slower
7. Star Schema
8. Snowflake Schema
9. Storage Efficiency
10. Complexity
11. Query Performance

