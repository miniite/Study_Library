---
type: "practice"
difficulty: "Deep"
---

# How would you design a partitioned table for daily sales data to keep costs low?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ BigQuery Loading and Table Design](<../../../../TOOLS/Google_Cloud/BigQuery/BigQuery_Loading_and_Table_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a cost-effective partitioned table in BigQuery for daily sales data, we need to consider several factors including storage costs, query performance, and administrative overhead. We'll use date-based partitioning to leverage BigQuery's pricing model, which charges based on the amount of data scanned rather than stored.

#### Triaging the Question

The core of this question is to understand how to optimize cost while maintaining efficient querying capabilities for daily sales data. BigQuery partitions tables by date, and we will leverage this feature to reduce costs. Key assumptions include:

The sales data is time-series.
We are concerned with storing and querying historical data up to a certain period (e.g., 1 year).
Queries typically focus on recent data.

#### First Principles Solution

BigQuery automatically partitions tables by partition key, which in this case will be the date part of the sales data timestamp. By setting the appropriate partitioning strategy, we can ensure that only relevant data is scanned during queries, thereby reducing costs and improving performance.

Step 1: Define the Table Schema

We first define a table schema that includes necessary fields such as sales_id, date, amount, etc.

SQL
CREATE TABLE sales (
  sales_id INT64,
  date DATE,
  amount DECIMAL(38, 2),
  product_id INT64,
  location STRING,
  ...
)
PARTITION BY DATE(date);
Step 2: Data Ingestion and Partitioning Strategy

To ensure efficient partitioning:

Ensure the date field is included in the schema.

#### Cons
ider rolling up old data into a table with coarser granularity after a certain period to save costs.
Step 3: Query Optimization

Queries often focus on recent data. We can use BigQuery's query optimization features to target specific partitions based on date ranges.

SQL
SELECT * FROM sales WHERE date >= '2023-01-01' AND date <= '2023-06-30';

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, partitioning by date ensures that only relevant data is scanned during queries. This approach reduces costs by minimizing the amount of data processed and improves query performance. However, it's crucial to manage the lifecycle of old data to avoid unnecessary storage costs.

#### Common mistakes
include:

Failing to define an appropriate partition key.
Not considering the trade-offs between partitioning and querying.
Ignoring the cost implications of older data that is rarely queried.

#### Follow-ups
might involve discussing how to handle large volumes of daily data, optimizing for specific query patterns, or implementing automated data lifecycle management using BigQuery's capabilities.

**Code:**

```sql
CREATE TABLE sales (
  sales_id INT64,
  date DATE,
  amount DECIMAL(38, 2),
  product_id INT64,
  location STRING,
  ...
)
PARTITION BY DATE(date);
```

**Example input / output:**

```text
SELECT * FROM sales WHERE date >= '2023-01-01' AND date <= '2023-06-30';
```

**Flow:**

1. Yes
2. No
3. Data Ingestion
4. Partition by Date?
5. Create Partitioned Table
6. Use Coarser Granularity forOld Data
7. Query Optimization
8. Reduce Costs, ImprovePerformance

