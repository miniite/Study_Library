---
type: "practice"
difficulty: "Deep"
---

# An index scan is chosen but random I/O on an HDD-backed volume makes it slower than a seq scan. How do you get the planner to prefer the seq scan without a global cost change?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Index-only Scans](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Index-only_Scans.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this scenario where the PostgreSQL query planner prefers an index scan over a sequential scan despite poor performance due to random I/O on HDD, we need to ensure that the planner chooses the most efficient access method based on local context and cost metrics. This involves understanding how the query costs are calculated and adjusting them locally without changing global settings.

The key idea here is leveraging PostgreSQL's cost-based optimizer by modifying the estimated costs of the index scan and sequential scan operations in a way that reflects their true performance characteristics, thus guiding the planner to choose the more efficient seq scan.

#### Triage
The question revolves around understanding how PostgreSQL decides between different execution plans for queries. Specifically, we need to manipulate the internal cost estimates to reflect the actual I/O costs of an HDD-backed storage system. Assumptions include that the sequential scan is faster due to reduced random I/O and that we can modify query costs locally without affecting other parts of the database.

Step-by-step from first principles:

Identify Cost Estimation: PostgreSQL uses a cost model to determine the best execution plan for each query. The cost estimation includes CPU time, I/O operations (both read and write), and other factors.
Understand the Current Cost Calculation: In this case, the planner is estimating that an index scan will be faster due to lower cost per operation compared to a sequential scan, despite actual poor performance on HDD.
Adjust Local Costs: We need to adjust the cost estimates locally for this specific query to reflect the true I/O costs of an HDD. This can be done using SET LOCAL commands or by creating custom statistics that accurately represent the I/O characteristics.

#### Solution

Use SET LOCAL: The most straightforward approach is to use SET LOCAL within the query execution context:

SQL
SET LOCAL random_page_cost TO 2;
SET LOCAL seq_page_cost TO 0.1;

-- Your query here
SELECT * FROM large_table WHERE id = 12345;
Here, random_page_cost is set to a higher value (e.g., 2), which increases the estimated cost of an index scan due to HDD I/O.
seq_page_cost is set to a lower value (e.g., 0.1), indicating that sequential scans are faster.

Custom Statistics: Another approach involves creating custom statistics for the table:

SQL
-- Create or update statistics with accurate page cost values
ANALYZE large_table WITH (
  RANDOM_PCT = 50,
  PAGE_COST = (2 / 32768),
  TUPLE_COST = 1/8192,
  SEQ_PAGE_COST = 0.1
);
These parameters help PostgreSQL understand the I/O characteristics better and adjust its cost estimates accordingly.

#### Closing Summary
By adjusting local costs using SET LOCAL or custom statistics, we can guide the planner to prefer a sequential scan when dealing with HDD-backed storage systems. This approach ensures that the query execution plan aligns with actual performance metrics without globally changing the database's cost model.

#### Common Mistakes/Follow-ups

Overlooking global settings: Modifying costs locally does not affect global settings, but using SET commands can lead to unexpected behavior if used inconsistently.
Underestimating page costs: Incorrectly setting PAGE_COST or SEQ_PAGE_COST could result in overly optimistic cost estimates and suboptimal query plans.
Ignoring other factors: While adjusting I/O costs is crucial, other factors like CPU time should also be considered to ensure comprehensive optimization.

**Code:**

```sql
-- Example of SET LOCAL usage
SET LOCAL random_page_cost TO 2;
SET LOCAL seq_page_cost TO 0.1;

SELECT * FROM large_table WHERE id = 12345;
```

