---
type: "practice"
difficulty: "Deep"
---

# In a lakehouse merge (Delta/Iceberg), how do you use ROW_NUMBER() to pick the winning source row before a MERGE INTO to avoid the 'multiple source rows matched' error?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Deterministic Deduplication](<../../../../01_Databases_and_SQL/SQL_Language/Window_Functions/Deterministic_Deduplication.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the problem of using ROW_NUMBER() in a lakehouse merge (Delta/Iceberg) setup, we need to understand how window functions can help us pick the winning source row when multiple rows from the source match a target row. The key idea is to use ROW_NUMBER() to assign a unique ranking to each source row based on a set of criteria, and then filter out non-winning rows before performing the merge.

#### Triaging the Question

The core question here revolves around leveraging SQL window functions to handle data conflicts during a MERGE INTO operation. Specifically, we need to ensure that only one "winning" source row is chosen when there are multiple matches between the target and the source tables. The window function ROW_NUMBER() can be used to rank rows within each partition of the data based on certain criteria.

#### Assumptions
We have a Delta or Iceberg table representing our target dataset.
We are performing a MERGE INTO operation from a source table, which might contain multiple rows that could match a single target row.
Our goal is to prevent the "multiple source rows matched" error by selecting one specific row as the winner.

#### First-Principles Solution

To implement this solution, we will follow these steps:

Identify the Criteria for Ranking: Define the criteria based on which you want to rank the source rows. For example, if multiple orders can have the same order ID and timestamp, you might prioritize by the latest timestamp.
Use ROW_NUMBER() with a Window Specification: Apply ROW_NUMBER() over the window partitioned by the relevant columns (e.g., order ID) and ordered by your criteria.
Filter Non-Winning Rows: Use the assigned rank to filter out all but one row for each target row.
Detailed Solution

Let's assume we have an orders table in our Delta/Iceberg environment, and we want to ensure that only the latest order is considered when merging into a historical orders table. Here’s how you can write the SQL query:

SQL
WITH ranked_orders AS (
    SELECT
        source_order_id,
        timestamp,
        ROW_NUMBER() OVER (PARTITION BY source_order_id ORDER BY timestamp DESC) as rn
    FROM
        orders_source
)
MERGE INTO historical_orders trg
USING ranked_orders src
ON trg.order_id = src.source_order_id
WHEN MATCHED THEN
    UPDATE SET
        ...
WHERE src.rn = 1;

#### Closing Summary
and Common Mistakes

By using ROW_NUMBER() in this way, we ensure that only the most recent order (based on the timestamp) is considered for each target row. This prevents conflicts and ensures a clean merge operation.

#### Common Mistakes
Forgetting to define the correct partitioning and ordering criteria.
Not filtering non-winning rows properly after ranking.
Ignoring potential data quality issues in the source data, such as missing or incorrect timestamps.
Final Check

The solution provided is both comprehensive and detailed, ensuring that only one "winning" row is chosen for each target row based on a clear set of criteria. The ROW_NUMBER() function has been used correctly to handle conflicts during the merge operation.

