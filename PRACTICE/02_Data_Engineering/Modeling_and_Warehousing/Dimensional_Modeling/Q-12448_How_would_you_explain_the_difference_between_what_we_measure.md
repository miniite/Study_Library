---
type: "practice"
difficulty: "Basic"
---

# How would you explain the difference between 'what we measure' and 'how we slice it' in a warehouse?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Additive and Allocated Measures](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Additive_and_Allocated_Measures.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In data warehousing, "what we measure" refers to the raw metrics or dimensions that are collected and stored, while "how we slice it" pertains to how these measurements are aggregated, filtered, and analyzed in reports or queries. Let's break down each concept first principles.

#### Triaging the Question

The question is essentially asking you to distinguish between the data elements (measures) and their usage patterns (granularity and aggregation). This understanding is critical for designing effective star and snowflake schemas.

#### Assumptions
You are familiar with basic concepts of relational databases.
Star Schema involves fact tables and dimension tables.
Snowflake Schema further normalizes dimension tables.
The question implicitly assumes familiarity with data warehouse design principles.

#### First Principles Solution

What We Measure (Measures/Metrics):

These are the primary business metrics or KPIs that we track in our database, such as sales revenue, customer count, transaction amount, etc.
In a star schema, these measures are stored directly within the fact table.

How We Slice It (Granularity and Aggregation):

This refers to how the data is grouped, aggregated, and queried at different levels of detail.
For example, in sales data, we might aggregate by day, week, month, or year depending on our reporting needs.
In a snowflake schema, this could involve breaking down dimensions into more granular tables.
Illustration

Imagine a scenario where you are designing a data warehouse for an e-commerce company. Here's how the concepts apply:

Measuring Sales: You have raw transaction data with columns like transaction_id, product_id, customer_id, and total_amount.

This is "what we measure."

Slicing Sales Data:

Aggregating sales by day, week, month, or year.
Filtering sales based on specific products, customers, or time periods.
Calculating metrics like total revenue, average transaction value, and customer lifetime value.

#### Closing Summary

Understanding the difference between "what we measure" and "how we slice it" is crucial for designing effective data models. Measures are the core business KPIs stored in the fact table, while slicing involves how these measures are aggregated and queried to provide meaningful insights.

#### Common Mistakes/Follow-Ups
Mistake: Over-normalizing dimensions can lead to complex queries and slower performance.

#### Follow-Up
Discussing whether star or snowflake schema is more appropriate based on the query patterns and data volume.

**Comparison:**

| transaction_id | product_id | customer_id | total_amount | transaction_date |
| customer_id | name          | email                     |
| product_id | category_id | name           | description  |

**Code:**

```sql
-- Example of a simple fact table in Star Schema
CREATE TABLE sales (
    transaction_id INT,
    product_id INT,
    customer_id INT,
    total_amount DECIMAL(10, 2),
    transaction_date DATE
);

-- Example of dimension tables in Snowflake Schema
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(150)
);

CREATE TABLE products (
    product_id INT PRIMARY KEY,
    category_id INT,
    name VARCHAR(200),
    description TEXT
);
```

**Example input / output:**

```text
Input (Query):
SELECT
    SUM(total_amount) AS total_revenue,
    DATE_TRUNC('week', transaction_date) AS week_of_transaction,
    products.name AS product_name
FROM sales
JOIN customers ON sales.customer_id = customers.customer_id
JOIN products ON sales.product_id = products.product_id
GROUP BY week_of_transaction, product_name

Output:
+------------------+------------+--------------+
| total_revenue    | week_of_transaction | product_name |
+------------------+------------+--------------+
| 110.67           | 2023-10-01        | T-shirt      |
| 75.68            | 2023-10-02        | Jeans        |
+------------------+------------+--------------+
```

**Data:**

```json
{
  "sales": [
    {"transaction_id": 1, "product_id": 101, "customer_id": 501, "total_amount": 34.99, "transaction_date": "2023-10-01"},
    {"transaction_id": 2, "product_id": 102, "customer_id": 502, "total_amount": 75.68, "transaction_date": "2023-10-02"}
  ],
  "customers": [
    {"customer_id": 501, "name": "John Doe", "email": "john.doe@example.com"},
    {"customer_id": 502, "name": "Jane Smith", "email": "jane.smith@example.com"}
  ],
  "products": [
    {"product_id": 101, "category_id": 1, "name": "T-shirt", "description": "White cotton T-shirt"},
    {"product_id": 102, "category_id": 2, "name": "Jeans", "description": "Dark blue jeans"}
  ]
}
```

**Flow:**

1. Sales Data
2. Star Schema
3. Fact Table
4. Dimension Tables
5. Customers
6. Products
7. Sum of Total Amount
8. Date Truncation to Week
9. Product Name Aggregation

