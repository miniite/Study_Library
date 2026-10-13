---
type: "practice"
difficulty: "Deep"
---

# How would you tune a query that joins a partitioned fact table to a dimension where partition pruning is not happening because the filter is on the dimension not the fact?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ EXISTS Semi-joins and Predicate Placement](<../../../../01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries/EXISTS_Semi-joins_and_Predicate_Placement.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To optimize a query involving a partitioned fact table and a dimension, where partition pruning is failing due to filtering conditions applied only to the dimension, we need to ensure that the database can effectively use the partitions in the fact table based on the join condition. This involves rethinking how the filters are applied and possibly restructuring the query or indexes.

#### Triage

The core issue here is that the database isn't pruning partitions correctly because the filter conditions on the dimension don't involve any partitioning key from the fact table. To address this, we can:

Reframe the Query: Ensure that the filtering condition involves both tables.
Indexing: Create or adjust indexes to facilitate partition pruning.
Query Restructuring: Use subqueries or derived tables if necessary.

#### First-Principles Solution

Understand Partitioning: Partitioning in databases divides a large table into smaller, more manageable parts based on ranges of values (e.g., dates). The key here is that partition pruning can only occur when the filter condition includes the partition column.

Refactor Filters: Modify the query so that filters applied to the dimension also involve the partition key in the fact table.

Create or Adjust Indexes: Ensure there are indexes on both tables that support efficient joins and filtering, especially those involving the partition keys.

Query Optimization: Use derived tables or subqueries if needed to ensure the optimizer can apply filters early in the query execution plan.

Analyze Execution Plan: Examine the query execution plan with tools like EXPLAIN to identify where pruning is failing and make necessary adjustments.

Step-by-Step Solution

Identify Partition Key: Determine which columns are used for partitioning in both tables.

Ensure Filter Involvement: Modify the filtering condition so it involves the fact table's partition key. For example, if your dimension has a date column and the fact table is partitioned by the same date range:

SQL
WHERE d.date BETWEEN 'start_date' AND 'end_date' AND f.partition_column = 'partition_value'

Create or Adjust Indexes: Add indexes on columns used in join conditions and filters to speed up access:

SQL
CREATE INDEX idx_fact_partition ON fact_table (partition_column, other_columns);
CREATE INDEX idx_dim_on_partition ON dimension_table (date_column, other_columns);

Use Derived Tables/Subqueries: If necessary, restructure the query using derived tables or subqueries to ensure filters are applied early in execution:

SQL
WITH filtered_dimension AS (
    SELECT * FROM dimension WHERE date BETWEEN 'start_date' AND 'end_date'
)
SELECT * FROM fact_table f JOIN filtered_dimension d ON f.date = d.date;

Analyze Execution Plan: Use EXPLAIN to analyze the execution plan and identify bottlenecks:

SQL
EXPLAIN SELECT * FROM fact_table f JOIN dimension d ON f.date = d.date WHERE d.date BETWEEN 'start_date' AND 'end_date';

#### Closing Summary
and Common Mistakes

By ensuring that filters involve both tables, creating appropriate indexes, and analyzing the execution plan, we can significantly improve query performance by enabling partition pruning. Common mistakes include:

Not involving the fact table's partition key in filtering conditions.
Overlooking the importance of indexing on relevant columns.
Failing to analyze and optimize the execution plan.
Self-Check

The solution is comprehensive and covers all necessary steps for optimizing the query with respect to partition pruning.

**Comparison:**

| Fact Table Columns | Description |
|-------------------|-------------|
| partition_column  | Date range used for partitioning |
| date              | Join key with dimension table |
| Dimension Table Columns | Description |
|-------------------------|-------------|
| date                    | Partition key and join key with fact table |

**Code:**

```sql
-- Example SQL Queries
WITH filtered_dimension AS (
    SELECT * FROM dimension WHERE date BETWEEN '2023-01-01' AND '2023-12-31'
)
SELECT * FROM fact_table f JOIN filtered_dimension d ON f.date = d.date;
```

**Example input / output:**

```text
-- Input
EXPLAIN SELECT * FROM fact_table f JOIN dimension d ON f.date = d.date WHERE d.date BETWEEN 'start_date' AND 'end_date';

-- Output
"Partition pruning not applied, full scan may be happening."
```

**Data:**

```json
{
  "dimension": {
    "date": "2023-06-15",
    "other_columns": "value"
  },
  "fact_table": [
    {
      "partition_column": "202306",
      "date": "2023-06-15",
      "other_columns": "value"
    }
  ]
}
```

**Flow:**

1. No
2. Yes
3. No
4. Yes
5. Query
6. Filter Condition?
7. Full Scan
8. Involves Partition Key?
9. Add Derived Table/Subquery
10. Apply Filters Early
11. Create Indexes on RelevantColumns
12. Analyze Execution Plan withEXPLAIN
13. Optimization Not Needed
14. Indexes Created
15. Improved Performance

