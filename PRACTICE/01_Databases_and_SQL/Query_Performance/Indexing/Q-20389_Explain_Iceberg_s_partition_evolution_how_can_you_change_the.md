---
type: "practice"
difficulty: "Expert"
---

# Explain Iceberg's partition evolution: how can you change the partitioning of a huge table without rewriting old data, and how does pruning still work?

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ Partitioning and Data Skipping in Lakes](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Partitioning_and_Data_Skipping_in_Lakes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview

Iceberg is an open-source storage layer that provides advanced features like ACID transactions, time travel queries, and dynamic partitioning in Hadoop. One of its key strengths is the ability to change table partitions without rewriting old data, a crucial feature for maintaining performance as your schema evolves over time.

#### Triaging the Question

The question revolves around understanding how Iceberg handles changes in partitioning schemes for large tables. Specifically:

How can you modify the partitioning structure of an existing table?
What happens to old data when the partitioning is changed, and how does pruning still function effectively?

#### First-Principles Solution
Understanding Iceberg's Partition Evolution

Iceberg uses a dynamic partitioning model that allows for flexible changes in schema without needing to rewrite all historical data. This flexibility comes from its metadata layer, which stores information about the table structure, partitions, and file locations.

Key Idea: To change the partitioning of an existing large table in Iceberg:

Create a new version of the table with the updated schema.
Use incremental operations to repartition data into the new schema while keeping old data intact.
Step-by-Step Process

Schema Evolution:

Define the new partitioning scheme.
Create a new version of the table schema in Iceberg metadata.

Data Repartitioning:

Use Iceberg's ALTER TABLE commands to add or remove partitions.
Perform incremental data repartitioning operations, ensuring old data remains unaltered.

Pruning:

Ensure that pruning works effectively by leveraging the metadata layer to filter out unnecessary files during queries.

#### Implementation
Example
SQL
-- Step 1: Evolve schema with new partitioning
ALTER TABLE my_table SET TBLPROPERTIES ('partitioning'='new_partition_column');

-- Step 2: Repartition data into new partitions
INSERT INTO my_table PARTITION(new_partition_column) SELECT ...;

-- Step 3: Prune old files (if necessary)
DELETE FROM my_table WHERE condition;
Detailed Walkthrough

Schema Evolution:

Suppose you have a table my_table with an existing partitioning schema based on year, month, and day.
You decide to introduce a new column region for better data organization.
SQL
ALTER TABLE my_table SET TBLPROPERTIES ('partitioning'='new_region, year, month, day');

Data Repartitioning:

Use incremental operations to repartition the existing data into the new schema.
SQL
INSERT INTO my_table PARTITION(new_region, year, month, day) SELECT region, EXTRACT(YEAR FROM date_column), EXTRACT(MONTH FROM date_column), EXTRACT(DAY FROM date_column) FROM source_table;

Pruning:

Ensure pruning works effectively by leveraging the metadata layer.
SQL
DELETE FROM my_table WHERE condition AND new_partition_column IS NULL; -- Prune old files based on new partitioning criteria

#### Closing Summary

By carefully managing schema evolution and incremental repartitioning, Iceberg allows for changes in partitioning without rewriting historical data. This is achieved through a combination of metadata management and controlled data movement operations.

#### Common Mistakes
and Follow-Ups

Mistake: Forgetting to maintain the integrity of old data during schema changes.

#### Follow-Up
Always use incremental operations and verify that old data remains unaltered until fully repartitioned.

Mistake: Improper pruning leading to unnecessary file management overhead.

#### Follow-Up
Ensure proper metadata filtering during queries to avoid redundant workloads.

**Comparison:**

| Step | Action | Description |
|------|--------|-------------|
| 1    | Schema Evolution | Define and apply the updated partitioning schema. |
| 2    | Data Repartitioning | Incrementally move existing data into new partitions. |
| 3    | Pruning | Filter out unnecessary files using metadata-driven queries. |

**Code:**

```sql
-- Example schema evolution command
ALTER TABLE my_table SET TBLPROPERTIES ('partitioning'='new_region, year, month, day');

-- Example repartitioning command
INSERT INTO my_table PARTITION(new_region, year, month, day) SELECT region, EXTRACT(YEAR FROM date_column), EXTRACT(MONTH FROM date_column), EXTRACT(DAY FROM date_column) FROM source_table;

-- Example pruning command
DELETE FROM my_table WHERE condition AND new_partition_column IS NULL;
```

**Example input / output:**

```text
-- Initial schema evolution
ALTER TABLE my_table SET TBLPROPERTIES ('partitioning'='new_region, year, month, day');

-- Incremental repartitioning
INSERT INTO my_table PARTITION(new_region, year, month, day) SELECT region, EXTRACT(YEAR FROM date_column), EXTRACT(MONTH FROM date_column), EXTRACT(DAY FROM date_column) FROM source_table;

-- Pruning old files
DELETE FROM my_table WHERE condition AND new_partition_column IS NULL;
```

**Data:**

```json
{
  "table_name": "my_table",
  "partition_columns": ["new_region", "year", "month", "day"],
  "source_table": {
    "columns": ["region", "date_column"]
  }
}
```

**Flow:**

1. Schema Evolution
2. ALTER TABLE my_table SETTBLPROPERTIES
3. Data Repartitioning
4. INSERT INTO my_tablePARTITION(...)
5. Pruning
6. DELETE FROM my_tableWHERE ... ANDnew_partition_column IS NULL

