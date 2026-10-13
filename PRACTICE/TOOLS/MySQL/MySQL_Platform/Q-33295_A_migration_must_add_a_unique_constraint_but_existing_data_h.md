---
type: "practice"
difficulty: "Deep"
---

# A migration must add a unique constraint but existing data has duplicates — walk through detecting, deciding a merge/dedup policy, and applying it without downtime.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Online Schema Changes](<../../../../TOOLS/MySQL/MySQL_Platform/Online_Schema_Changes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview

In this scenario, we need to apply a unique constraint to an existing table in MySQL while handling duplicate data that already exists. The key steps involve detecting duplicates, deciding on a merge/dedup policy, and then applying the changes without causing downtime.

#### Triage

The core question here is how to add a unique constraint to a table with existing duplicate entries in a way that minimizes disruption. We need to:

Detect Duplicates: Identify rows that violate the uniqueness constraint.
Policy Decision: Decide on a merge/dedup policy (e.g., update one value, ignore duplicates).
Apply Changes: Implement the changes without causing downtime.

#### Assumptions
The table has multiple columns and rows with possible duplicate entries based on certain columns.
There are no immediate business-critical operations running on this table that require a strict uptime guarantee.
We can afford some temporary read/write lock during the process, but not full downtime for the entire application.
Step-by-Step Solution
1. Detect Duplicates

To detect duplicates, we can use GROUP BY with aggregate functions or common table expressions (CTEs). For example:

SQL
WITH duplicates AS (
    SELECT column1, column2, COUNT(*) as count
    FROM your_table
    GROUP BY column1, column2
    HAVING count > 1
)
SELECT * FROM duplicates;

This query will return all rows that have duplicate entries based on column1 and column2.

2. Policy Decision

Once we have identified the duplicates, we need to decide how to handle them:

Merge: Update one of the conflicting rows.
Ignore: Delete or mark those rows as no longer valid.

For example, if the policy is to merge:

SQL
UPDATE your_table t1 JOIN (
    SELECT column1, column2, MAX(id) AS max_id
    FROM your_table
    GROUP BY column1, column2
    HAVING COUNT(*) > 1
) t2 ON t1.column1 = t2.column1 AND t1.column2 = t2.column2
SET t1.some_column = t2.some_column;

This query updates the some_column of one row in each set of duplicates, keeping the latest ID.

3. Apply Changes Without Downtime

To apply changes without downtime, we can use a combination of database transactions and temporary tables:

Create a Temporary Table: Use CREATE TABLE AS SELECT to copy data from the original table into a new one.
Apply Merges: Perform merge operations on the temporary table.
Switch Tables: Rename or swap the names of the old and new tables.

Here’s an example:

SQL
-- Step 1: Create a temporary table
CREATE TABLE temp_table AS SELECT * FROM your_table;

-- Step 2: Apply merges on the temporary table
WITH duplicates AS (
    SELECT column1, column2, MAX(id) as max_id
    FROM temp_table
    GROUP BY column1, column2
    HAVING COUNT(*) > 1
)
UPDATE temp_table t JOIN duplicates d ON t.column1 = d.column1 AND t.column2 = d.column2 SET t.some_column = (SELECT some_column FROM your_table WHERE id = d.max_id);

-- Step 3: Switch tables
RENAME TABLE your_table TO old_table, temp_table TO your_table;

-- Clean up the old table if needed
DROP TABLE old_table;

#### Closing Summary

In summary, we first detect duplicates using GROUP BY, decide on a merge policy, and then use transactions and temporary tables to apply these changes without downtime. This approach ensures minimal disruption while maintaining data integrity.

#### Common Mistakes/Follow-Ups
Not Handling Transactions Properly: Ensure all operations are wrapped in a transaction to maintain atomicity.
Overlooking Data Integrity: Always validate the merge/dedup policy to ensure no critical information is lost or altered.
Downtime Risk: Even with temporary tables, there's still a risk of brief downtime during the renaming operation. Consider using database-specific features like partitioning or sharding if full uptime is crucial.

**Comparison:**

| Column | Data Type | Description |
|--------|----------|-------------|
| id     | INT      | Unique ID   |
| column1| VARCHAR  | First Key   |
| column2| VARCHAR  | Second Key  |
| some_column| TEXT    | Some Value  |

**Code:**

```sql
WITH duplicates AS (
    SELECT column1, column2, COUNT(*) as count
    FROM your_table
    GROUP BY column1, column2
    HAVING count > 1
)
SELECT * FROM duplicates;

CREATE TABLE temp_table AS SELECT * FROM your_table;

WITH duplicates AS (
    SELECT column1, column2, MAX(id) as max_id
    FROM temp_table
    GROUP BY column1, column2
    HAVING COUNT(*) > 1
)
UPDATE temp_table t JOIN duplicates d ON t.column1 = d.column1 AND t.column2 = d.column2 SET t.some_column = (SELECT some_column FROM your_table WHERE id = d.max_id);

RENAME TABLE your_table TO old_table, temp_table TO your_table;

DROP TABLE old_table;
```

**Example input / output:**

```text
-- Initial State
SELECT * FROM your_table;

-- Detect Duplicates
WITH duplicates AS (
    SELECT column1, column2, COUNT(*) as count
    FROM your_table
    GROUP BY column1, column2
    HAVING count > 1
)
SELECT * FROM duplicates;

-- Merge Policy Applied to Temporary Table
CREATE TABLE temp_table AS SELECT * FROM your_table;

WITH duplicates AS (
    SELECT column1, column2, MAX(id) as max_id
    FROM temp_table
    GROUP BY column1, column2
    HAVING COUNT(*) > 1
)
UPDATE temp_table t JOIN duplicates d ON t.column1 = d.column1 AND t.column2 = d.column2 SET t.some_column = (SELECT some_column FROM your_table WHERE id = d.max_id);

-- Switch Tables and Clean Up
RENAME TABLE your_table TO old_table, temp_table TO your_table;

DROP TABLE old_table;
```

**Data:**

```json
{
  "your_table": {
    "columns": ["id", "column1", "column2", "some_column"],
    "data": [
      {"id": 1, "column1": "A", "column2": "B", "some_column": "value1"},
      {"id": 2, "column1": "C", "column2": "D", "some_column": "value2"},
      {"id": 3, "column1": "A", "column2": "B", "some_column": "value3"}
    ]
  }
}
```

**Flow:**

1. Detect Duplicates
2. Policy Decision
3. Merge/Dedup Policy
4. Apply Merges on Temp Table
5. Rename Tables to ApplyChanges
6. Cleanup Old Table

