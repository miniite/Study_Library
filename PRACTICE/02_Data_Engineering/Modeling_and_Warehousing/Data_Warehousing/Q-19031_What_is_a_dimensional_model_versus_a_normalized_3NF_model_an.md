---
type: "practice"
difficulty: "Deep"
---

# What is a dimensional model versus a normalized (3NF) model and when is each appropriate?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Distribution Keys](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Data_Warehousing/Distribution_Keys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview

In data warehousing, we often need to balance the trade-offs between ease of querying and storage efficiency. Two common modeling approaches are dimensional models and normalized (3NF) models. Dimensional models provide a star or snowflake schema focused on fast query performance, while 3NF models offer better data integrity but may sacrifice some query performance due to more complex joins.

#### Triage

The question asks for the distinction between dimensional and normalized models in data warehousing and their respective applicability scenarios. The interviewer is likely probing your understanding of the core differences, the implications on query performance and storage, and how these models impact real-world business processes.

#### First-Principles Solution
Dimensional Model

A dimensional model typically uses a star or snowflake schema where data is stored in fact tables surrounded by dimension tables. The key characteristics are:

Star Schema: A central fact table with multiple dimension tables (e.g., time, geography, product).
Snowflake Schema: Still centers around the fact table but organizes dimensions hierarchically to reduce redundancy.
Simplicity and Performance: Ideal for reporting and analytics because queries are straightforward, using fewer joins.
Normalized Model

A normalized model adheres to normalization rules (typically 3NF) where data is stored in multiple tables with minimal redundancy:

Reduced Redundancy: Each piece of information is stored only once.
Data Integrity: Ensures that each row uniquely identifies a record, reducing the risk of inconsistencies.
Complexity and Performance: Requires more complex joins to retrieve related data, which can slow down reporting queries.
When to Use Each

Dimensional Model:

Applicability: Best for business intelligence (BI) applications where fast query performance is critical. Common in retail, finance, and healthcare.

#### Example Scenario
A sales report that needs to be generated quickly across multiple dimensions like time, region, product category.

Normalized Model:

Applicability: Suitable for transactional systems and operational databases where data integrity and consistency are paramount. Less focused on real-time reporting.

#### Example Scenario
An e-commerce platform’s inventory management system that updates frequently with transactions; ensuring no duplicate entries is crucial.

#### Closing Summary

In summary, dimensional models are optimized for fast querying and business intelligence tasks, while normalized models prioritize data integrity and reduce redundancy at the cost of slower query performance. The choice between them depends on the specific needs and constraints of your use case.

#### Common Mistakes/Follow-Ups
Misunderstanding Normalization Levels: Often confusing 1NF (first normal form) with 3NF, which is not directly relevant to dimensional modeling.
Overlooking Query Performance in Dimensional Models: Not recognizing that while they offer speed, they can still have performance issues if improperly designed or used for large datasets.
Ignoring Business Requirements: Failing to consider whether the model aligns with business needs and constraints.
Triaging Follow-Ups

The interviewer might probe deeper on:

Join Strategies in Dimensional Models - Discuss how denormalization can be managed without compromising performance too much.
Integration Between Models - Explore scenarios where both models are used together, e.g., a dimensional model for reporting and a normalized one for operational data.
Performance Optimization Techniques - Talk about techniques like indexing, partitioning, and materialized views that can enhance the performance of either model.

**Comparison:**

```
Sales Fact
Product
Time
Region
Categories
```

**Code:**

```sql
-- Example Star Schema (Star)
CREATE TABLE sales_fact (
    sale_id INT PRIMARY KEY,
    product_id INT,
    region_id INT,
    time_id DATE,
    quantity_sold INT,
    price DECIMAL(10,2),
    FOREIGN KEY (product_id) REFERENCES products(id),
    FOREIGN KEY (region_id) REFERENCES regions(id),
    FOREIGN KEY (time_id) REFERENCES times(date)
);

CREATE TABLE products (
    id INT PRIMARY KEY,
    name VARCHAR(255),
    category_id INT,
    FOREIGN KEY (category_id) REFERENCES categories(id)
);

CREATE TABLE regions (
    id INT PRIMARY KEY,
    region_name VARCHAR(255),
    country_id INT,
    FOREIGN KEY (country_id) REFERENCES countries(id)
);

-- Example Snowflake Schema
CREATE TABLE sales_fact (
    sale_id INT PRIMARY KEY,
    product_id INT,
    time_id DATE,
    quantity_sold INT,
    price DECIMAL(10,2),
    FOREIGN KEY (product_id) REFERENCES products(id),
    FOREIGN KEY (time_id) REFERENCES times(date)
);

CREATE TABLE product (
    id INT PRIMARY KEY,
    name VARCHAR(255),
    category_id INT
);

CREATE TABLE categories (
    id INT PRIMARY KEY,
    category_name VARCHAR(255),
    FOREIGN KEY (id) REFERENCES products(category_id)
);
```

**Example input / output:**

| Input (Star Schema) | Output (Sales Report for September 2023) |
|---------------------|-----------------------------------------|
| sales_fact          | Region-wise Total Sales                   |
| products            | Product-wise Average Price                |
| categories          | Category-wise Quantity Sold               |
| Input (Snowflake Schema) | Output (Product Details by Category)    |
|--------------------------|-----------------------------------------|
| sales_fact               | Product name, category, and region      |
| product                  | Detailed product info                    |
| categories               | Hierarchical category structure         |

**Data:**

```json
{
  "sales_fact": [
    {
      "sale_id": 101,
      "product_id": 1001,
      "time_id": "2023-09-01",
      "quantity_sold": 50,
      "price": 12.5
    },
    {
      "sale_id": 102,
      "product_id": 1002,
      "time_id": "2023-09-01",
      "quantity_sold": 75,
      "price": 15.0
    }
  ],
  "products": [
    {
      "id": 1001,
      "name": "Widget A",
      "category_id": 401
    },
    {
      "id": 1002,
      "name": "Gadget B",
      "category_id": 401
    }
  ],
  "categories": [
    {
      "id": 401,
      "category_name": "Tools"
    }
  ]
}
```

**Flow:**

1. Normalized Model
2. Dimensional Model
3. Query Execution
4. Star Schema
5. Snowflake Schema
6. Dimension Tables
7. Fact Table

