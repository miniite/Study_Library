---
type: "practice"
difficulty: "Basic"
---

# What is a data warehouse and how is it different from an operational (transactional) database?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Kimball Approach](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Kimball_Approach.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A data warehouse is essentially a centralized repository designed to support business intelligence, reporting, and analytics. In contrast, an operational or transactional database supports day-to-day operations, handling real-time transactions and updates. The key differences lie in their design principles, purpose, and how they handle data.

#### Triaging the Question

The question asks for definitions of a data warehouse and operational (transactional) database, highlighting their distinct roles and architectures. It's important to understand that while both store data, the context and usage are fundamentally different.

#### Assumptions
The database developer is familiar with basic concepts of databases.
The interviewer might probe on design choices and trade-offs between these two types of databases.

#### First-Principles Solution

Operational Database:

Purpose: Real-time transaction processing, supporting business operations.
Design Principles:
ACID (Atomicity, Consistency, Isolation, Durability) properties to ensure data integrity in real-time transactions.
Normalization to minimize redundancy and improve update consistency.
Scales horizontally for handling high write loads.

Data Warehouse:

Purpose: Business intelligence, reporting, and analytics.
Design Principles:
Designed for read-heavy workloads with minimal updates (batch loads).
Star or Snowflake schema to denormalize data for faster querying and analysis.
Optimized for complex queries involving large datasets.
Detailed Explanation

Operational Database:

Think of an operational database as the backbone of a business. It handles all day-to-day transactions, ensuring that every sale, update, or transaction is recorded accurately and in real-time.

#### Example
A customer management system where every user interaction needs to be logged with timestamps and updated immediately.

Data Warehouse:

Imagine a data warehouse as a comprehensive database designed for analysis. It collects historical data from various sources and stores it in a way that makes querying large datasets more efficient.

#### Example
An online retailer collecting sales data, customer interactions, and product information over months or years to run complex analyses on customer behavior.

#### Closing Summary

In summary, the primary difference between an operational database and a data warehouse lies in their design goals. Operational databases are optimized for write-heavy operations with ACID properties, while data warehouses focus on read-heavy workloads, using star/snowflake schemas to optimize query performance for analytics.

#### Common Mistakes/Follow-Ups

Mistake: Equating operational and data warehousing concerns too closely.

#### Follow-Up
Clarify the differences in design principles and use cases.

Mistake: Not understanding that normalization is generally a principle of operational databases, but data warehouses often require denormalization for better query performance.

#### Follow-Up
Discuss when to normalize versus denormalize data based on the specific use case.

**Comparison:**

```
Customer	Product	Sale Date
John Doe	Laptop	2023-10-01
Customers	Products	Sales Facts
John Doe	Laptop	2023-10-01
```

**Code:**

```sql
-- Example schema for an operational database (simplified)
CREATE TABLE sales (
    id INT PRIMARY KEY,
    customer_id INT NOT NULL,
    product_id INT NOT NULL,
    sale_date TIMESTAMP NOT NULL
);

-- Example schema for a data warehouse using Star Schema
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(255)
);

CREATE TABLE products (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(255)
);

CREATE TABLE sales_facts (
    sale_id INT PRIMARY KEY,
    customer_id INT NOT NULL,
    product_id INT NOT NULL,
    sale_date TIMESTAMP NOT NULL
);
```

**Example input / output:**

```text
Input:
- Insert a new sale in the operational database

Output:
- Sales table updated with sale details

Input:
- Load historical sales data into a data warehouse

Output:
- Data added to the star schema (customers, products, sales_facts) for analysis
```

**Data:**

```json
{
  "customer": {
    "id": 100,
    "name": "John Doe",
    "email": "john.doe@example.com"
  },
  "product": {
    "id": 200,
    "name": "Laptop"
  },
  "sale": {
    "id": 300,
    "customer_id": 100,
    "product_id": 200,
    "sale_date": "2023-10-01T14:00:00Z"
  }
}
```

**Flow:**

1. ACID properties
2. Real-time updates
3. Optimized for queries
4. Denormalization
5. Operational Database
6. Write Operations
7. Ensure Data Integrity
8. High Write Load
9. Data Warehouse
10. Read Operations
11. Faster Analysis
12. Less Redundant Data

