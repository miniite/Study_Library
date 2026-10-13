---
type: "practice"
difficulty: "Basic"
---

# What are table statistics and why does the query optimizer depend on them?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Statistics and Sargability](<../../../../01_Databases_and_SQL/Query_Performance/Execution_Plans/Statistics_and_Sargability.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Table statistics play a crucial role in database performance, particularly when it comes to optimizing queries. In simple terms, they provide the query optimizer with valuable information about the data distribution and cardinality of tables, allowing it to make more informed decisions on how to execute the query efficiently.

To understand why the query optimizer depends on these statistics, let's break down their importance from first principles:

#### Triaging the Question

The question is essentially asking for an explanation of what table statistics are and why they are so critical. The core idea here is that without accurate statistics, the database system cannot make optimal decisions about the execution plan for a query.

Key Assumptions and Context
Table Statistics: These are metadata records that provide statistical information about the data in a table.
Query Optimizer: This component of the database engine determines the most efficient way to execute a SQL query based on various factors like indexes, data distribution, and cardinality.

#### First Principles Solution

Definition of Table Statistics:

Row Count: The total number of rows in the table.
Histograms: Frequency distributions of values for columns with high cardinality (e.g., unique values).
Average Row Length: The average size of a row, which helps in estimating the space required for operations like scans or inserts.

Purpose:

Cardinality Estimation: Accurate statistics allow the optimizer to estimate how many rows will match certain conditions in the query (e.g., WHERE clause).
Cost-Based Decision Making: The optimizer uses these estimates to calculate the cost of different execution plans and choose the one with the lowest estimated cost.

Why the Query Optimizer Depends on Them:

Efficiency: Without accurate statistics, the optimizer might choose an inefficient plan (e.g., a full table scan instead of using an index).
Scalability: In large databases, making incorrect assumptions can lead to performance bottlenecks.
Dynamic Nature: As data changes over time, so do the statistics. Regular updates are necessary to maintain accurate information.
Detailed Example

#### Cons
ider a simple example where we have a Sales table with columns like ProductID, Region, and QuantitySold. If the database has up-to-date statistics:

The optimizer can quickly estimate that only a small percentage of rows might match WHERE Region = 'West'.
It will likely choose an index on Region to speed up this query.

Without accurate statistics, it might assume that all regions are equally represented and opt for a full table scan, which would be much slower.

#### Common Mistakes
and Follow-Ups
Ignoring Statistics Updates: Failing to regularly update statistics can lead to outdated information.
Over-Reliance on Indexes: Sometimes, the optimizer might choose an index that is not as efficient as expected due to inaccurate cardinality estimates.
Complex Queries: For complex queries with multiple conditions, incorrect statistics can significantly impact performance.

#### Closing Summary

In conclusion, table statistics are essential for effective query optimization because they provide the necessary information to make informed decisions about execution plans. Accurate and up-to-date statistics ensure that database systems can deliver efficient performance, especially in large or frequently updated datasets.

**Comparison:**

```
Column	Description
Row Count	Total number of rows in the table.
Histograms	Frequency distributions for columns with high cardinality.
Average Row Length	Estimated average size of a row.
```

**Code:**

```sql
-- Example of creating a histogram manually (for demonstration purposes)
CREATE STATISTICS Histogram ON Sales.ProductID;
```

**Example input / output:**

```text
- **Input**: `SELECT * FROM Sales WHERE ProductID = 1 AND Region = 'West'`
- **Output**: Optimized query plan using index on `ProductID` and `Region`.
```

**Data:**

```json
{
  "Sales": {
    "ProductID": {"histogram": [{"value": 1, "frequency": 500}, {"value": 2, "frequency": 300}, ...]},
    "Region": {"cardinality": 4},
    "QuantitySold": {"average_row_length": 20}
  }
}
```

**Flow:**

1. Yes
2. No
3. Query Received
4. Parse Query
5. Is Table Updated?
6. Get Statistics
7. Estimate Cardinality and Cost
8. Choose Execution Plan
9. Execute Plan
10. Update Statistics First

