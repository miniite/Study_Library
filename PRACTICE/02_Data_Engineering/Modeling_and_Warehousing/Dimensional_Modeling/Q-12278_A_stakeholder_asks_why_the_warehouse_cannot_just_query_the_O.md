---
type: "practice"
difficulty: "Intermediate"
---

# A stakeholder asks why the warehouse cannot just query the OLTP tables directly; how do you explain the dimensional-modeling payoff with concrete trade-offs?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ OLTP vs Star Schema Performance](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/OLTP_vs_Star_Schema_Performance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address a stakeholder's concern about querying OLTP (Online Transaction Processing) tables directly instead of using a star or snowflake schema in data warehousing, we need to explain the dimensional-modeling benefits and trade-offs involved.

#### Triage

The core question here is why data warehouses require a specific structure like star or snowflake schemas. We should clarify that querying OLTP (transactional) databases directly can lead to performance issues due to frequent updates, lack of normalization, and complex queries. Dimensional modeling optimizes for query performance by denormalizing the data into more efficient structures. Let's explore these concepts step-by-step.

#### First-Principles Solution
1. Understanding OLTP vs Data Warehouse

OLTP (Online Transaction Processing) systems are designed for frequent, transaction-oriented operations like updates and inserts. They typically use normalized tables to ensure data integrity and consistency. In contrast, Data Warehouses are optimized for analytical queries that aggregate large volumes of historical data.

2. Performance Considerations
Query Performance: OLTP databases are not optimized for complex analytical queries, which can lead to slow response times.
Concurrency Issues: Frequent updates in OLTP tables can cause locks and deadlocks, affecting query performance.
Data Normalization: While normalization is crucial for transactional integrity, it can make queries more complex and slower.
3. Dimensional Modeling Benefits

Star Schema:

Fact Tables: Store measurements or metrics.
Dimension Tables: Contain detailed attributes (e.g., dates, customers, products).
Join Efficiency: Simplifies joins through surrogate keys, improving query performance.

Snowflake Schema:

Starlike Core: A central fact table with dimension tables connected to it.
Normalized Dimensions: Dimensions can be further decomposed into sub-dimensions, reducing data redundancy and improving update efficiency.
4. Trade-offs
Data Redundancy: Star or snowflake schemas may introduce some redundancy but significantly improve query performance.
Maintenance Overhead: Adding more tables increases the maintenance overhead but is manageable with proper schema design and indexing strategies.
Scalability: Properly designed dimensional models can scale better for large datasets compared to OLTP databases.

#### Closing Summary

In conclusion, while OLTP databases are optimized for transactional operations, data warehouses benefit from dimensional modeling to improve query performance and handle complex analytical queries more efficiently. The trade-offs include increased redundancy but are justified by the performance gains in an environment that primarily supports reporting and analytics.

#### Common Mistakes/Follow-Ups
Ignoring Normalization: Some might argue that complete denormalization is not necessary, as a balance between normalization and denormalization can be struck.
Overlooking Concurrency Issues: Not addressing potential concurrency issues in OLTP environments can lead to suboptimal solutions.

**Comparison:**

```
Sales
Fact Table
Products(Dimension)
Customers(Dimension)
Categories(Dimension)
```

**Code:**

```sql
-- Example of a Star Schema Table Creation
CREATE TABLE sales (
    sale_id INT PRIMARY KEY,
    product_id INT,
    customer_id INT,
    sale_date DATE,
    quantity INT,
    price DECIMAL(10, 2)
);

CREATE TABLE products (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100),
    category_id INT
);

CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    customer_name VARCHAR(100),
    address VARCHAR(255)
);
```

**Example input / output:**

```text
Input:
SELECT
    p.product_name,
    c.customer_name,
    s.quantity * s.price AS total_price
FROM
    sales s
JOIN
    products p ON s.product_id = p.product_id
JOIN
    customers c ON s.customer_id = c.customer_id;

Output:
+------------+-------------------+---------------+
| product_name | customer_name     | total_price   |
+------------+-------------------+---------------+
| Laptop      | John Doe          | 249.95        |
+------------+-------------------+---------------+
```

**Data:**

```json
{
  "sales": [
    {
      "sale_id": 1,
      "product_id": 101,
      "customer_id": 202,
      "sale_date": "2023-09-15",
      "quantity": 5,
      "price": 49.99
    }
  ],
  "products": [
    {
      "product_id": 101,
      "product_name": "Laptop",
      "category_id": 301
    }
  ],
  "customers": [
    {
      "customer_id": 202,
      "customer_name": "John Doe",
      "address": "123 Main St, Anytown"
    }
  ]
}
```

**Flow:**

1. +-----------------------+
2. | OLTP vs Data Warehouse|
3. +----------+-----------+
4. |
5. v
6. +----------+-----------+
7. | Query    | Performance|
8. +----------+-----------+
9. | OLTP     | Slow      |
10. |          |           |
11. | Data     | Fast      |
12. | Warehouse|           |
13. +----------+-----------+
14. |
15. v
16. +----------+-----------+
17. | Concurrency | Locks    |
18. |            | Deadlocks|
19. +----------+-----------+

