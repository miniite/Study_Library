---
type: "practice"
difficulty: "Intermediate"
---

# Why does SELECT * cost more than selecting specific columns in BigQuery?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ BigQuery Cost Control](<../../../../TOOLS/Google_Cloud/BigQuery/BigQuery_Cost_Control.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In BigQuery, SELECT * is generally more expensive compared to specifying specific columns because it involves querying a larger amount of data and potentially storing unnecessary information during intermediate stages. This difference can be significant in terms of cost and performance.

#### Triaging the Question
What the Question Is Asking: The question probes your understanding of BigQuery's internal operations, specifically how selecting all columns (SELECT *) affects the query execution plan compared to selecting specific columns.
Key Assumptions:
Understanding of BigQuery’s cost model and data processing.
Awareness of how intermediate results are stored in query execution.
Reasoning from First Principles: We'll break down the process step by step, explaining why SELECT * is more resource-intensive.
Step-by-Step Solution:

Data Retrieval:

When you use SELECT *, BigQuery has to fetch and transfer all columns of a table for every row.
This increases the data size that needs to be transferred over the network, which can significantly impact performance and cost, especially for large tables.

Intermediate Results Storage:

In query execution plans, intermediate results are often stored in temporary tables or materialized views. When you select all columns (SELECT *), these intermediate results contain more data, leading to higher storage costs.
For example, if the original table has 10 columns and each row is 1 KB, SELECT * will transfer 10 KB per row, whereas selecting specific columns might only require transferring 5 KB.

Resource Utilization:

Query execution consumes CPU and memory resources. Fetching more data (with SELECT *) means that the query engine has to process a larger amount of data, increasing the load on these resources.
This can also lead to longer query execution times, as the system needs more time to scan through and transfer all columns.

Cost Implications:

BigQuery charges based on the amount of data processed (including intermediate results) and the duration of the query execution.
Therefore, SELECT * will incur higher costs due to increased data volume and longer processing times.

#### Closing Summary

In summary, SELECT * in BigQuery is more costly than selecting specific columns because it retrieves and processes a larger amount of data. This can lead to higher network transfer costs, increased storage usage for intermediate results, and greater resource utilization by the query engine. It's always advisable to specify only the necessary columns to optimize performance and reduce costs.

#### Common Mistakes/Follow-Ups
Mistake: Overlooking the impact of SELECT * on intermediate results.

#### Follow-Up
Discussing how query optimization techniques can help manage these costs, such as using partitioning or clustering strategies to improve query performance.

**Comparison:**

| Column | Type    |
|--------|---------|
| column1 | INT     |
| column2 | STRING  |
| column3 | FLOAT   |

**Code:**

```sql
-- Example SQL for selecting specific columns vs all columns
-- Specific Columns
SELECT column1, column2 FROM your_table;

-- All Columns (SELECT *)
SELECT * FROM your_table;
```

**Example input / output:**

```text
-- Input (SELECT all columns)
SELECT * FROM your_table;

-- Output (Intermediary result storage might be larger due to all columns being present)
+----+----------------+-----+
| id |     name       | age |
+----+----------------+-----+
| 1  | John Doe       | 25  |
| 2  | Jane Smith     | 30  |
+----+----------------+-----+

-- Input (SELECT specific columns)
SELECT column1, column2 FROM your_table;

-- Output (Intermediary result storage might be smaller due to fewer columns being present)
+----+----------------+
| id |     name       |
+----+----------------+
| 1  | John Doe       |
| 2  | Jane Smith     |
+----+----------------+
```

**Data:**

```json
{
  "table_schema": {
    "columns": [
      {"name": "column1", "type": "INT"},
      {"name": "column2", "type": "STRING"},
      {"name": "column3", "type": "FLOAT"}
    ]
  },
  "rows": [
    { "column1": 1, "column2": "data1", "column3": 0.5 },
    { "column1": 2, "column2": "data2", "column3": 1.0 }
  ]
}
```

**Flow:**

1. SELECT *
2. Fetch and transfer all columns
3. High network cost, higherstorage usage for intermediateresults
4. Increased resource utilizationby query engine
5. SELECT specific columns
6. Fetch and transfer onlynecessary columns
7. Lower network cost, lowerstorage usage for intermediateresults
8. Lower resource utilization byquery engine

