---
type: "practice"
difficulty: "Deep"
---

# You must add a NOT NULL column with a default to a 5TB InnoDB table on MySQL 8 in production. Walk through what the engine actually does per version and how you'd avoid a metadata-lock storm.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Migrations with Read Your Writes](<../../../../04_Data_Architecture_and_Strategy/Platform_Strategy/Schema_Evolution/Migrations_with_Read_Your_Writes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When adding a NOT NULL column with a default value to an existing InnoDB table on MySQL 8, we need to understand the underlying mechanics of the operation, particularly in terms of how the engine handles metadata locks and data migrations. This process involves several steps that must be carefully managed to avoid locking bottlenecks (metadata lock storms) and ensure the integrity of the data.

#### Triaging the Question

The core issue here is ensuring a smooth and efficient migration without causing significant performance degradation or downtime. The key points are:

Understanding the Engine's Behavior: MySQL 8 uses InnoDB, which employs row-level locking for NOT NULL columns.
Avoiding Metadata Lock Storms: Multiple transactions trying to modify the same table concurrently can lead to contention and bottlenecks.
Strategic Migrations: We need a plan that minimizes the impact on existing operations while ensuring data integrity.

#### First-Principles Solution
Identify Existing Constraints: Ensure no conflicting constraints exist (e.g., foreign keys, triggers).
Add Column with Default: Use ALTER TABLE to add the column with a default value.
Optimize Data Population: Populate the new column in batches or during off-peak hours.
Drop Old Columns if Necessary: Once data is populated and validated, drop old columns.
Step-by-Step Solution

Check Current Schema:

SQL
DESCRIBE your_table_name;

Add Column with Default Value:

SQL
ALTER TABLE your_table_name ADD COLUMN new_column_name DATATYPE DEFAULT default_value NOT NULL;
This command adds a new column and sets it to NOT NULL with a default value.

Populate New Column in Batches: To avoid overwhelming the database, populate the new column in batches or during off-peak hours.

SQL
-- Batch 1: Update first 10% of rows
UPDATE your_table_name SET new_column_name = default_value WHERE id >= (SELECT MIN(id) FROM your_table_name LIMIT 999998, 1);

-- Repeat for other batches until all rows are updated.

Ensure Data Integrity:

Run CHECK TABLE to validate the integrity of the new column.
SQL
CHECK TABLE your_table_name;

Optimize and Re-index if Necessary: After data population, optimize the table for performance.

SQL
OPTIMIZE TABLE your_table_name;

Validate Data: Ensure all rows have valid values in the new column before proceeding.

SQL
SELECT COUNT(*) FROM your_table_name WHERE new_column_name IS NULL OR new_column_name = default_value;

Drop Old Columns if Necessary: Once data is validated, drop old columns that are no longer needed.

SQL
ALTER TABLE your_table_name DROP COLUMN old_column_name;

#### Closing Summary
and Common Mistakes/Follow-Ups

This strategy ensures a smooth addition of the NOT NULL column with minimal impact on existing operations. Key mistakes to avoid include:

Not validating data before dropping old columns.
Failing to consider concurrent operations during data population.

Common follow-ups might involve discussing alternative strategies, such as using triggers or temporary tables for more complex migrations.

**Comparison:**

| Operation       | SQL Statement                   |
|-----------------|--------------------------------|
| Add Column      | `ALTER TABLE ...`               |
| Update Rows     | `UPDATE your_table_name SET ...`|
| Check Integrity | `CHECK TABLE your_table_name;`  |
| Optimize Table  | `OPTIMIZE TABLE your_table_name;`|

**Code:**

```sql
-- Add column with default value
ALTER TABLE your_table_name ADD COLUMN new_column_name DATATYPE DEFAULT default_value NOT NULL;

-- Batch update rows
UPDATE your_table_name SET new_column_name = default_value WHERE id >= (SELECT MIN(id) FROM your_table_name LIMIT 999998, 1);

-- Check table integrity
CHECK TABLE your_table_name;

-- Optimize and re-index
OPTIMIZE TABLE your_table_name;

-- Validate data
SELECT COUNT(*) FROM your_table_name WHERE new_column_name IS NULL OR new_column_name = default_value;

-- Drop old columns if necessary
ALTER TABLE your_table_name DROP COLUMN old_column_name;
```

**Example input / output:**

```text
Input:
- Existing table: your_table_name with 5TB data.

Output:
- New column added: new_column_name.
- Data population in batches completed successfully.
- Table integrity validated.
- Old column dropped if necessary.
```

**Data:**

```sql
-- Example schema
CREATE TABLE your_table_name (
    id INT PRIMARY KEY,
    existing_column1 VARCHAR(255),
    -- ... other columns ...
);
```

**Flow:**

1. Yes
2. Yes
3. No
4. Check Current Schema
5. Execute 'ALTER TABLE...'?
6. Populate New Column inBatches
7. Validate Data Integrity
8. Optimize and Re-index?
9. Drop Old Columns
10. Operation Complete

