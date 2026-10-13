---
type: "practice"
difficulty: "Deep"
---

# Why can a query with a highly selective filter still do a full scan if the index is very unselective?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Query Rewrites and Plan Stability](<../../../../01_Databases_and_SQL/Query_Performance/Execution_Plans/Query_Rewrites_and_Plan_Stability.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with database queries, it's important to understand that execution plans and query tuning involve multiple factors beyond just selectivity. A query might perform a full table scan even when there’s a highly selective filter using an unselective index due to several reasons related to cost-based optimization and performance trade-offs.

Firstly, let’s briefly summarize the core concept: Execution plans in databases are determined by the optimizer, which estimates costs based on statistics like cardinality (number of distinct values) and selectivity. A full table scan can be chosen over an index because the optimizer might decide that the cost of accessing the index is higher than scanning the entire table.

#### Triage

The question probes your understanding of how database optimizers make decisions during query execution, particularly focusing on the balance between using indexes versus performing full scans. It also tests your ability to consider various factors in choosing an efficient execution plan. Key assumptions include that the database optimizer uses cost-based optimization and has accurate statistics about table cardinality.

Step-by-Step Solution from First Principles

Understanding Selectivity vs. Cardinality:

Selectivity: This refers to how well a filter narrows down the result set. A highly selective filter means it will return a small subset of rows.
Cardinality: This measures the distinct number of values in a column.

Cost-Based Optimization:

Databases use cost-based optimizers (CBO) that estimate the cost of different execution plans and choose the plan with the lowest estimated cost. The CBO considers factors like CPU, I/O, and memory costs.
For instance, accessing an index might involve reading a B-tree or hash structure, which could have higher overhead than scanning a large table.

#### Example Scenario

Suppose you have a users table with 10 million rows. The column age has a high cardinality (distinct ages) but is not indexed.
A query filters users aged between 25 and 30 years old, which could be highly selective.

Optimizer Decision:

If the optimizer estimates that scanning all 10 million rows (full table scan) would still result in fewer disk I/O operations or lower CPU overhead compared to reading an unselective index, it might choose a full table scan.
This decision can also depend on the actual data distribution and statistics maintained by the database.

Additional Factors:

Index Usage Costs: Even if an index exists, accessing it requires additional overhead such as fetching rows from disk based on pointer values stored in the index nodes.
Buffer Cache Hit Rate: If there are many cache misses, reading a full table might be faster than repeatedly hitting the disk for index accesses.

#### Closing Summary
+ Common Mistakes

In summary, while a highly selective filter might suggest using an index, the database optimizer can still choose a full table scan based on estimated costs and performance trade-offs. Key factors include index usage overhead, buffer cache hit rates, and accurate cardinality statistics maintained by the database.

#### Common mistakes
include assuming that a selectivity-based decision is always correct without considering cost estimates or performance implications. Always validate decisions with actual execution plans and performance metrics.

**Comparison:**

| Column | Cardinality |
|--------|-------------|
| age    | 4           |

**Code:**

```sql
-- Example SQL query
SELECT * FROM users WHERE age BETWEEN 25 AND 30;
```

**Example input / output:**

```text
Input:
- SQL Query: SELECT * FROM users WHERE age BETWEEN 25 AND 30;

Output:
- Execution Plan: Full Table Scan (Estimated Cost: 100)
- Actual Execution: Full Table Scan because buffer cache hit rate is low and index usage overhead is high.
```

**Data:**

```json
{
  "table": {
    "name": "users",
    "columns": ["id", "name", "age"],
    "cardinality": 10_000_000,
    "distinctValuesAge": [20, 30, 40, 50]
  }
}
```

**Flow:**

1. Yes
2. No
3. If Full Table Scan Lower Cost
4. Otherwise
5. Query
6. Is Index Selective?
7. Use Index
8. Estimate Cost of Full TableScan and Index Access
9. Perform Full Table Scan
10. Use Index

