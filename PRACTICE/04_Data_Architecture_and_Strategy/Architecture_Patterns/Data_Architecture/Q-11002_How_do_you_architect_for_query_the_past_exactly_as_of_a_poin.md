---
type: "practice"
difficulty: "Expert"
---

# How do you architect for 'query the past exactly as of a point in time' across a lakehouse where dimension tables are SCD Type 2 and fact tables are append-only? Walk me through the join logic and where naive as-of joins produce silently wrong historical reports.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Real-time OLAP Serving Layer](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/Real-time_OLAP_Serving_Layer.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address querying the past exactly as of a point in time across a lakehouse architecture, we need to ensure that our dimension tables (SCD Type 2) and fact tables (append-only) are correctly joined using an as-of join logic. This ensures that the latest version of each dimension record is used for queries at any given historical timestamp. Here’s how you would architect this:

#### Overview
The core idea is to implement a robust as-of join mechanism, which involves tracking the effective time periods in SCD Type 2 dimensions and ensuring that joins respect these temporal constraints.

#### Triaging the Question

What is Being Probed: The interviewer wants to see how you handle complex temporal data querying within a lakehouse architecture.

#### Assumptions
You are working with a hybrid data storage approach where both historical (lake) and transactional (warehouse) data coexist. Fact tables have an append-only schema, while dimension tables use SCD Type 2 for handling changes in historical records.

#### First-Principles Solution

Step-by-Step Explanation:
Temporal Tracking: Each dimension record should be associated with a start and end date (effective_from, effective_to) indicating its validity period.
As-of Join Logic: For any given query timestamp, you need to join the fact table with the appropriate version of each dimension. This involves:
Identifying which versions of dimensions are valid as of the specified timestamp.
Performing a LEFT JOIN on the fact table with the dimension table, using the latest valid version based on the query’s time constraint.

#### Example Scenario

Suppose we have a fact table sales_fact and a dimension table product_dim (SCD Type 2).
We want to retrieve sales data for all products as of June 1st, 2023.
The query would look something like:
SQL
SELECT f.*, d.*
FROM sales_fact f
LEFT JOIN (
  SELECT p.*, MAX(p.effective_to) AS max_effective_date
  FROM product_dim p
  WHERE p.effective_from <= '2023-06-01' AND (p.effective_to > '2023-06-01' OR p.effective_to IS NULL)
  GROUP BY p.product_id, p.effective_from
) d ON f.product_id = d.product_id AND f.timestamp <= d.max_effective_date;

#### Common Mistakes
and Follow-Ups:

Mistake 1: Naively using a JOIN without considering the temporal constraints can lead to incorrect results.

#### Follow-Up
Questions:
How do you handle rolling updates or continuous changes in dimensions?
What performance optimizations can be applied to this join logic?

**Comparison:**

```
table "sales_fact" as f {
  columns: [product_id, timestamp, quantity]
}
table "product_dim" as d {
  columns: [product_id, product_name, effective_from, effective_to]
}
join f ON d WHERE f.timestamp <= d.max_effective_date
```

**Code:**

```sql
-- Example SQL for as-of join
SELECT f.*, d.*
FROM sales_fact f
LEFT JOIN (
  SELECT p.product_id, MAX(p.effective_to) AS max_effective_date
  FROM product_dim p
  WHERE p.effective_from <= '2023-06-01' AND (p.effective_to > '2023-06-01' OR p.effective_to IS NULL)
  GROUP BY p.product_id, p.effective_from
) d ON f.product_id = d.product_id AND f.timestamp <= d.max_effective_date;
```

**Example input / output:**

```text
Input:
Sales Fact Table: sales_fact (product_id, timestamp, quantity)
Product Dimension Table: product_dim (product_id, product_name, effective_from, effective_to)

Query: SELECT f.*, d.* FROM sales_fact f LEFT JOIN ... as of 2023-06-01

Output:
+---------+------------+--------+------------------+-------------+
| product_id | timestamp   | quantity| product_name      | effective_to|
+---------+------------+--------+------------------+-------------+
|     101  | 2023-05-01 |    10   | Widget A          | null        |
|     102  | 2023-06-01 |    20   | Widget C          | null        |
+---------+------------+--------+------------------+-------------+
```

**Data:**

```json
{
  "sales_fact": [
    {"product_id": 101, "timestamp": "2023-05-01", "quantity": 10},
    {"product_id": 102, "timestamp": "2023-06-01", "quantity": 20}
  ],
  "product_dim": [
    {"product_id": 101, "product_name": "Widget A", "effective_from": "2022-01-01", "effective_to": null},
    {"product_id": 102, "product_name": "Widget B", "effective_from": "2023-01-01", "effective_to": "2023-05-31"},
    {"product_id": 102, "product_name": "Widget C", "effective_from": "2023-06-01", "effective_to": null}
  ]
}
```

**Flow:**

1. Query for a specific timestamp
2. Filter Fact Table by Timestamp
3. Join with Product DimensionTable
4. Select Latest Version of EachDimension
5. Retrieve Joined Data withCorrect Dimensions
6. Output Historical Reports

