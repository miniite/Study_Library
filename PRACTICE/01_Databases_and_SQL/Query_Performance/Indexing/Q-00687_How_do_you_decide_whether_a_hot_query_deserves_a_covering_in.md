---
type: "practice"
difficulty: "Deep"
---

# How do you decide whether a hot query deserves a covering index or a dedicated denormalized read table, factoring in team ownership, consistency needs, and long-term maintenance?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Covering Indexes and Index Strategy for Sharding](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/How_Indexes_Are_Used/Covering_Indexes_and_Index_Strategy_for_Sharding.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deciding between a covering index and a dedicated denormalized read table for handling hot queries, we must consider several factors including query performance, data consistency requirements, team ownership, and long-term maintainability. A key principle here is to balance the trade-offs between reducing the load on the primary database (through indexing) versus simplifying reads by storing pre-computed values in a separate table.

To begin, let's define what we mean by each:

Covering Index: An index that contains all or most of the columns needed for a query, thereby eliminating the need to fetch data from the actual table.
Dedicated Denormalized Read Table: A table created explicitly to serve read-heavy queries with pre-aggregated or denormalized data.

#### Triaging the Question

The core of this question revolves around understanding when it's more effective to use an advanced indexing strategy like covering or composite indexes, versus creating a new denormalized read table. The key factors are:

Query Patterns: How frequently and how complex the queries are.

#### Cons
istency Requirements: The level of freshness needed for data.
Team Ownership: The skill set and experience within the team.
Maintenance Costs: Long-term operational costs associated with maintaining the solution.

#### First Principles Solution

Query Analysis:

Frequency and Complexity: High-frequency, complex queries benefit most from covering indexes because they reduce read load significantly without needing to maintain a separate table.
Data Usage Patterns: If data is predominantly read in its entirety or subsets of columns are frequently accessed together, a composite index can be very effective.

#### Cons
istency Needs:

Real-time Consistency: For highly consistent data access, covering indexes ensure that the required data is always available and up-to-date.
Eventual Consistency: In scenarios where some staleness in data is acceptable, denormalized read tables can provide performance benefits without the overhead of maintaining indices.

Team Ownership:

Index Management Skills: Teams comfortable with indexing strategies might prefer covering indexes due to their familiarity and straightforward implementation.
Development Effort: Creating a denormalized table requires initial setup but can reduce ongoing development efforts, especially if complex queries are frequent.

Maintenance Costs:

Performance Overhead of Indexes: Covering indexes add overhead to insert/update operations since the index must be updated whenever data changes.
Read Performance Gains: On the other hand, denormalized read tables can significantly reduce read times but require more complex management and periodic updates.

#### Example Scenario

#### Cons
ider a scenario where an e-commerce application needs to frequently fetch product information along with reviews. The product table has several columns including product_id, name, price, category, etc., while the review table contains review_id, rating, timestamp, and other details linked to products.

Covering Index: A covering index on (product_id, name, price, category) could be sufficient if most queries only require these columns. However, if reviews are frequently accessed alongside product data, a composite index might not cover all necessary fields.

Denormalized Read Table: If the application needs to aggregate and serve reviews in real-time, creating a denormalized read table that pre-aggregates review ratings could be more effective. This approach reduces the load on the primary database during peak query times.

#### Closing Summary

In summary, whether to use a covering index or a dedicated denormalized read table depends on the specific needs of your application and the trade-offs you are willing to make. Covering indexes offer simplicity in implementation and maintainability but may introduce performance overhead during writes. On the other hand, denormalized tables provide high-performing reads at the cost of increased complexity in data management.

#### Common Mistakes/Follow-Ups
Over-reliance on Indexes: Not considering the write-side impact of index maintenance.
Ignoring Read Patterns: Assuming that a single type of index will solve all read performance issues without understanding the specific access patterns.
Underestimating Maintenance Costs: Failing to account for the ongoing effort required to maintain and update denormalized tables.

**Comparison:**

| Index Type | Purpose | Complexity | Maintenance |
| --- | --- | --- | --- |
| Covering Index | Reduces read load by including query columns in the index | Simple to implement | Increases write overhead |
| Denormalized Table | Simplifies reads and aggregates data for complex queries | Complex setup | Requires periodic updates |

**Code:**

```sql
-- Example of creating a covering index
CREATE INDEX idx_covering ON products(product_id, name, price, category);

-- Example of a denormalized read table
CREATE TABLE product_reviews (
    review_id INT,
    product_id INT,
    rating INT,
    timestamp TIMESTAMP,
    PRIMARY KEY (product_id)
);
```

**Example input / output:**

```text
-- Input for a query to fetch product details and reviews
SELECT p.product_id, p.name, p.price, r.rating
FROM products p
LEFT JOIN reviews r ON p.product_id = r.product_id

-- Output with covering index on (product_id, name, price)
p1: {"product_id": 1, "name": "Laptop", "price": 999.99, "rating": [4, 5]}

-- Output without covering index
p2: {"product_id": 1, "name": "Laptop", "price": 999.99}
```

**Data:**

```json
{
  "products": [
    {"product_id": 1, "name": "Laptop", "price": 999.99, "category": "Electronics"},
    {"product_id": 2, "name": "Smartphone", "price": 599.00, "category": "Electronics"}
  ],
  "reviews": [
    {"review_id": 1, "product_id": 1, "rating": 4, "timestamp": "2023-10-01"},
    {"review_id": 2, "product_id": 1, "rating": 5, "timestamp": "2023-10-02"}
  ]
}
```

**Flow:**

1. Query Analysis
2. Covering Index
3. Denormalized Read Table
4. Highly Consistent Data
5. Simple Queries
6. Real-time Aggregations
7. Complex Queries

