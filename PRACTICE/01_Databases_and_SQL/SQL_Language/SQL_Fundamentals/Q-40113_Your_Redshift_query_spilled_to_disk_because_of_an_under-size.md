---
type: "practice"
difficulty: "Deep"
---

# Your Redshift query spilled to disk because of an under-sized VARCHAR distribution. Explain DISTKEY and SORTKEY choice, and how a wrong DISTSTYLE causes broadcast or redistribution during a JOIN.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Large Values and TOAST](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Large_Values_and_TOAST.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand why your Redshift query spills to disk due to an under-sized VARCHAR distribution, we first need to delve into the concepts of DISTKEY, SORTKEY, and DISTSTYLE. The right choice here involves balancing data locality with efficient joins. Let's break it down step by step.

#### Triage
What is the Question Really Asking?

The question probes your understanding of how Redshift distributes and sorts data across nodes, particularly focusing on the impact of VARCHAR distributions on query performance. It’s asking you to explain the roles of DISTKEY, SORTKEY, and DISTSTYLE, and why choosing them incorrectly can lead to suboptimal query execution.

Key Assumptions
We are dealing with a Redshift cluster.
The table in question has a column with large VARCHAR values that is not properly indexed or distributed.
The query involves joins, which are the primary reason for the spill to disk issue.

#### First Principles Solution
1. Understanding DISTKEY, SORTKEY, and DISTSTYLE:
DISTKEY: Determines how data is partitioned across nodes in a Redshift cluster. Each node gets its own copy of the table, with rows distributed based on the value of the DISTKEY.
SORTKEY: Specifies which columns should be used to sort and further optimize queries involving those columns.
DISTSTYLE: Controls how data is distributed across nodes:
EVEN (default): Data is evenly spread across all nodes.
ALL: All data for the table goes on a single node, usually chosen based on SORTKEY.
KEY: Data is partitioned using the value of the DISTKEY.
2. Why an Under-Sized VARCHAR Distribution Can Cause Issues:
If you choose a column with large VARCHAR values as your DISTKEY, it can lead to poor data locality because:
Large strings take up more storage space, making them harder to hash and distribute evenly.
The distribution might become skewed if the strings have many unique values or long lengths.
3. Impact of a Wrong DISTSTYLE:
If you use EVEN or KEY as your DISTSTYLE, the query planner may need to perform a broadcast join, where data from multiple nodes is sent to every node during a JOIN. This can lead to excessive network traffic and spill to disk.
For example, if you have two tables with large VARCHAR distributions on different nodes, a JOIN operation might require redistributing all the data to one node, causing a broadcast join.

#### Example Scenario

Let's assume Table A has a column large_description VARCHAR(1024) as its DISTKEY. When performing a join between Table A and another table B (with a different distribution), Redshift might need to redistribute all rows of Table A across nodes due to the skewed nature of the VARCHAR values. This leads to broadcast joins, which can significantly impact performance and cause spills.

#### Closing Summary
+ Common Mistakes/Follow-Ups

To avoid these issues:

Choose columns with evenly distributed and non-skewed data as your DISTKEY.
Use SORTKEY for columns that are frequently queried or used in JOIN conditions.

#### Cons
ider using ALL distribution style for critical tables to ensure all necessary data is available on a single node.

#### Common mistakes
include:

Choosing large string fields as DISTKEY, leading to skewed distributions.
Using EVEN with large datasets, causing inefficient data movement during joins.
Not understanding the impact of different DISTSTYLE options on query performance.

For follow-ups, you might be asked about specific scenarios or how to fine-tune the distribution and sort keys for optimal performance in complex queries.

**Comparison:**

```
tableDiagram
    TABLE sales (id, product_name, sale_date, quantity_sold)
    TABLE orders (product_name, amount)

sales -->|JOIN| orders
```

**Code:**

```sql
-- Example of choosing DISTKEY and SORTKEY
CREATE TABLE sales (
    id INT,
    product_name VARCHAR(100),
    sale_date DATE,
    quantity_sold INT
) DISTSTYLE EVEN -- Use ALL if you expect frequent joins on `product_name`
                -- and want all data on a single node.
SORTKEY (sale_date, product_name);
```

**Example input / output:**

```text
-- Example input and output for a join operation with spill to disk
SELECT sa.id, sa.product_name, sa.sale_date, sa.quantity_sold, coalesce(co.amount, 0)
FROM sales sa
LEFT JOIN orders co ON sa.product_name = co.product_name -- Broadcast join due to large VARCHAR distribution
WHERE sale_date > '2023-01-01';
-- This query may spill to disk because of the large `VARCHAR` values and broadcast join.
```

**Data:**

```json
{
  "table": "sales",
  "columns": [
    { "name": "id", "type": "INT" },
    { "name": "product_name", "type": "VARCHAR(100)" },
    { "name": "sale_date", "type": "DATE" },
    { "name": "quantity_sold", "type": "INT" }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Sales Table
4. Is product_name DISTKEY?
5. Sort by sale_date,product_name
6. Choose better DISTKEY
7. Optimized for joins andqueries
8. Consider ALL or anotherstrategy

