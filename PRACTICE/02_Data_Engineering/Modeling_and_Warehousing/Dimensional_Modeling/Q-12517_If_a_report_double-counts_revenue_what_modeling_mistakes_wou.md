---
type: "practice"
difficulty: "Intermediate"
---

# If a report double-counts revenue, what modeling mistakes would you check first?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Additive and Allocated Measures](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Additive_and_Allocated_Measures.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In data engineering and specifically when dealing with data models like star or snowflake schemas, if a report is double-counting revenue, it's crucial to identify where the error might be occurring. At an intermediate level, we would check several key areas in our data model—mainly related to normalization levels, foreign keys, and potentially redundant dimensions.

First, let's triage what this question is probing: the ability to diagnose a common issue with financial reporting by understanding how data models can lead to such errors. We will need to consider potential misconfigurations in our schema that could cause duplication or incorrect summation of revenue data. For a fresher, we would focus on basic normalization issues and redundant relationships; for an intermediate engineer, we would delve into more advanced concepts like recursive joins and potentially hierarchical schemas; for a senior level, we might explore broader implications such as performance bottlenecks.

Step-by-Step Solution
Normalization Levels: Review the current schema to ensure that it is at the right normalization level. Double-counting often occurs in third-normal form (3NF) due to over-normalization where data should be denormalized for better reporting.
Foreign Keys and Dimensional Integrity: Check for any redundant or misplaced foreign keys. Ensure that each fact table has a direct relationship with all required dimensions, without unnecessary joins that might cause duplication.
Recursive Joins and Hierarchical Relationships: For a more complex schema like a snowflake, review the recursive nature of relationships between entities. Recursive joins can sometimes lead to over-counting if not handled carefully.
Duplicate Records: Verify for any duplicate records in dimension tables or fact tables that might contribute to double-counting.
Aggregation and Calculation Logic: Examine the aggregation logic used in reporting queries. Ensure that sums, averages, and other calculations are done correctly without introducing redundant counts.
Detailed Analysis
Normalization Levels

For a star schema, we typically have one or more fact tables surrounded by dimension tables. Over-normalizing can lead to issues where necessary data might be distributed across multiple tables, causing double-counting when aggregating. For example, if there's an intermediate fact table that should not exist, it might aggregate the same revenue twice.

Foreign Keys and Dimensional Integrity

Ensure that each fact table has a direct relationship with all relevant dimensions through proper foreign keys. Misplaced or redundant joins can lead to incorrect data being included in aggregated results. For instance, if there’s an unnecessary join between two dimension tables, this could introduce duplication.

Recursive Joins and Hierarchical Relationships

In a snowflake schema, hierarchical relationships are common but need careful handling. A recursive join might be used to get the full lineage of a product or customer, but it should not be overused as it can lead to redundant counts. For example, if a product's revenue is calculated by joining through multiple levels in the hierarchy without proper aggregation rules, this could result in double-counting.

Duplicate Records

Duplicate records are another common issue that can cause double-counting. Ensure that data cleaning processes are robust and remove any duplicates before aggregating. This includes checking for exact matches across tables or using unique identifiers to ensure each record is counted only once.

Aggregation Logic

Review the SQL queries used in reporting to make sure they correctly aggregate without introducing duplication. This involves understanding how joins, group by clauses, and other aggregation functions work together. For instance, a poorly written query might sum revenue multiple times if it includes redundant join conditions.

#### Closing Summary
+ Common Mistakes

#### Common mistakes
include:

Over-normalizing the schema leading to unnecessary complexity.
Misplaced or redundant foreign keys causing incorrect data inclusion.
Not properly handling hierarchical relationships with recursive joins.
Failing to clean and verify for duplicate records before aggregation.
Writing SQL queries that do not correctly handle group by clauses.

By systematically checking these areas, we can identify and fix the root causes of double-counting in revenue reports. This involves a combination of schema design review, data quality checks, and careful query writing.

**Comparison:**

| order_id | customer_id | order_date       |
|---------|------------|-----------------|
| 1       | 101        | 2023-05-01      |
| 2       | 102        | 2023-05-01      |
| order_id | product_id | quantity | price |
|---------|-----------|---------|-------|
| 1       | 101       | 2       | 100   |
| 2       | 101       | 3       | 100   |
| product_id | product_name |
|-----------|-------------|
| 101       | Laptop      |

**Code:**

```sql
-- Example of a poorly written SQL query that might lead to double-counting

SELECT
    p.product_name,
    SUM(o.quantity * i.price) as total_revenue
FROM
    orders o
JOIN
    order_items i ON o.order_id = i.order_id
JOIN
    products p ON i.product_id = p.product_id
JOIN
    customers c ON o.customer_id = c.customer_id
WHERE
    o.order_date BETWEEN '2023-01-01' AND '2023-06-30'
GROUP BY
    p.product_name;
```

**Example input / output:**

```text
-- Input
SELECT
    p.product_name,
    SUM(o.quantity * i.price) as total_revenue
FROM
    orders o
JOIN
    order_items i ON o.order_id = i.order_id
JOIN
    products p ON i.product_id = p.product_id
WHERE
    o.order_date BETWEEN '2023-01-01' AND '2023-06-30'
GROUP BY
    p.product_name;

-- Output
+------------------+----------------+
| product_name     | total_revenue  |
+------------------+----------------+
| Laptop           | 500            |
+------------------+----------------+
```

**Data:**

```json
{
  "orders": [
    {"order_id": 1, "customer_id": 101, "order_date": "2023-05-01"},
    {"order_id": 2, "customer_id": 102, "order_date": "2023-05-01"}
  ],
  "order_items": [
    {"order_id": 1, "product_id": 101, "quantity": 2, "price": 100},
    {"order_id": 2, "product_id": 101, "quantity": 3, "price": 100}
  ],
  "products": [
    {"product_id": 101, "product_name": "Laptop"}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Check Normalization Levels
6. Is Schema Over-normalized?
7. Review and Denormalize
8. Check Foreign Keys andDimensional Integrity
9. Are There Redundant Joins?
10. Remove Redundant Joins
11. Verify Duplicate Records
12. Check Aggregation Logic
13. Write Correct SQL Queries

