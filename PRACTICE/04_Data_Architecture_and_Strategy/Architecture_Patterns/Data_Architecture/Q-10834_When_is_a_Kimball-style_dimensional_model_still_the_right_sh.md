---
type: "practice"
difficulty: "Deep"
---

# When is a Kimball-style dimensional model still the right shape for ML feature serving versus a wide denormalized 'one big table' versus a mesh of domain feature products? What determined your answer at your last org?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Central vs Domain Feature Modeling](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Central_vs_Domain_Feature_Modeling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing data architecture for an organization, the choice between a Kimball-style dimensional model, a wide denormalized 'one big table,' and a mesh of domain feature products depends on several factors including the nature of the data, the query patterns, the storage requirements, and the performance needs. At my last organization, we evaluated these options based on our specific use cases and decided that a Kimball-style dimensional model was most suitable for our ML feature serving needs.

#### Triage

The question is essentially asking how to choose between different data architectures (dimensional models, wide tables, or mesh architectures) in the context of ML feature serving. The interviewer is probing understanding of data warehousing principles, query optimization, and the trade-offs between these architectural styles.

#### Assumptions
Use Case: We are focusing on a scenario where the primary goal is to serve features for machine learning models.
Data Volume & Velocity: Consider the scale at which data needs to be processed and served.
Query Patterns: Different query patterns require different types of schemas.

#### First-Principles Solution
Kimball-style Dimensional Model

A Kimball-style dimensional model, also known as a star or snowflake schema, is structured around fact tables and dimension tables. This architecture excels in providing fast read operations for analytical queries, making it highly suitable for reporting and business intelligence needs.

Why Use Kimball-style?

Fast Read Operations: Dimensional models are optimized for complex join operations, which are typical in ML feature serving.
Granular Control Over Data Access: Easier to enforce security policies and manage access to data.
Historical Consistency: Maintaining historical consistency is easier with a dimensional model.

When to Use Kimball-style?

High Query Complexity: Complex join operations are common in ML feature engineering, where features often require aggregations from multiple sources.
Security Requirements: Fine-grained access control and data lineage are important for maintaining data integrity and compliance.
Historical Data Retention: Need to maintain historical data for training models over time.
Wide Denormalized 'One Big Table'

A wide denormalized table, or a "one big table," is an approach where all the necessary fields for a particular use case are combined into a single table. This can simplify queries but often comes at the cost of data normalization and increased storage requirements.

Why Use One Big Table?

Simplicity in Querying: Easier to write and maintain simple queries, as all required data is in one place.
Reduced Joins: Fewer join operations are needed, which can improve performance for certain types of queries.

When to Use One Big Table?

Simplicity Over Performance: When the primary goal is to simplify query writing and maintenance, at the expense of potential storage overhead.
Limited Query Complexity: If most queries involve simple aggregations or lookups on a single dataset.
Storage Constraints: When storage is a limiting factor and denormalization can reduce the number of tables.
Mesh of Domain Feature Products

A mesh architecture involves breaking down data into smaller, more manageable pieces (feature products), each serving a specific domain or feature set. This approach enhances modularity and flexibility but can introduce complexity in managing interdependencies between features.

Why Use a Mesh?

Modularity: Easier to maintain and scale individual components.
Flexibility: Allows for rapid prototyping and experimentation with different feature sets without impacting the entire system.

When to Use a Mesh?

Modular Development: When the data architecture needs to support agile development practices, where features can be developed independently.
Scalability: For large-scale systems where different domains of expertise (e.g., user behavior, product usage) require their own isolated feature sets.

#### Closing Summary

At my last organization, we chose a Kimball-style dimensional model for ML feature serving due to the high complexity of our feature queries and the need for historical consistency. However, this decision is not one-size-fits-all, and the choice should be driven by specific requirements such as query patterns, storage constraints, and performance needs.

#### Common Mistakes/Follow-ups
Overlooking Query Complexity: Failing to account for complex join operations in ML feature engineering can lead to suboptimal data retrieval.
Neglecting Historical Data Retention: Ignoring the need for historical data in model training can result in poor generalization and performance degradation over time.
Ignoring Storage Constraints: Over-normalizing or under-normalizing can lead to either excessive storage costs or degraded performance, respectively.
Triaging the Last Organization's Decision

At my last organization, we faced a scenario where our ML models required complex feature engineering from multiple data sources. We opted for a Kimball-style dimensional model because it provided the necessary granularity and historical consistency without overcomplicating the system. However, we also considered a hybrid approach by creating smaller, more focused domain feature products for specific use cases.

**Comparison:**

| sales                          | products                  | customers                 |
|-------------------------------|---------------------------|---------------------------|
| sale_id, product_id, ...        | product_id, product_name, | customer_id, first_name,  |
|                                 | category_id               | last_name, email          |

**Code:**

```sql
-- Example of a star schema in SQL
CREATE TABLE sales (
    sale_id INT PRIMARY KEY,
    product_id INT,
    customer_id INT,
    store_id INT,
    sale_date DATE,
    quantity_sold INT,
    price DECIMAL(10, 2)
);

CREATE TABLE products (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(50),
    category_id INT
);

CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    email VARCHAR(100)
);
```

**Example input / output:**

```text
Input:
SELECT p.product_name, SUM(s.quantity_sold) as total_sales
FROM sales s
JOIN products p ON s.product_id = p.product_id
WHERE s.sale_date >= '2023-01-01' AND s.sale_date <= '2023-02-28'
GROUP BY p.product_name;

Output:
| product_name    | total_sales |
|-----------------|-------------|
| Laptop          | 5           |
| Smartphone      | 3           |
```

**Data:**

```json
{
  "sales": [
    {"sale_id": 1, "product_id": 101, "customer_id": 202, "store_id": 303, "sale_date": "2023-01-01", "quantity_sold": 5, "price": 29.99},
    {"sale_id": 2, "product_id": 102, "customer_id": 204, "store_id": 304, "sale_date": "2023-02-01", "quantity_sold": 3, "price": 19.99}
  ],
  "products": [
    {"product_id": 101, "product_name": "Laptop", "category_id": 1},
    {"product_id": 102, "product_name": "Smartphone", "category_id": 1}
  ],
  "customers": [
    {"customer_id": 202, "first_name": "John", "last_name": "Doe", "email": "john.doe@example.com"},
    {"customer_id": 204, "first_name": "Jane", "last_name": "Smith", "email": "jane.smith@example.com"}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Query Request
4. Is query complex?
5. (Kimball-style DimensionalModel)
6. (Wide Denormalized Table)
7. Retrieve Data
8. Retrieve Data
9. (Simplified Query Execution)
10. (Complex Join Operations)

