---
type: "practice"
difficulty: "Deep"
---

# How would you script a repeatable, forward-only migration for adding a column with a computed default backfill for 100M rows?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Online Schema Changes](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Online_Schema_Changes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this challenge of adding a new column with a computed default value to an existing table containing 100 million rows, we need to ensure that the migration is repeatable and forward-only. We'll focus on using DDL (Data Definition Language) for altering the schema and DML (Data Manipulation Language) for populating the data efficiently.

#### Triaging the Question

The question asks us to script a repeatable, forward-only migration process. This means we need to ensure that running the scripts multiple times won't alter the state of the database beyond what was intended initially. The "forward-only" aspect implies that once the new column is added and populated with default values, any future runs should not modify existing data.

#### Assumptions
We are using a relational database system (e.g., PostgreSQL, MySQL).
The table already exists and contains 100 million rows.
The computed default value for the new column can be derived from an existing column or set of columns in the same table.
The migration should handle large-scale data without causing significant performance impact.
Step-by-Step Solution
1. Back up the Database

Before making any changes, it’s crucial to back up the database to ensure that we have a fallback option if something goes wrong during the migration process.

SQL
-- Backup the database schema and data
pg_dump -U username -Fc -b -v mydatabase > backup.dump
2. Alter Table Schema to Add the New Column with Computed Default Value

We will add the new column using DDL, specifying that it has a computed default value.

SQL
-- Alter table schema
ALTER TABLE mytable ADD COLUMN new_column INT DEFAULT (existing_column + 1);
3. Validate the Migration Script for Correctness

To ensure the migration script works correctly, we can run it on a test environment or a small sample of data first.

SQL
-- Test the migration script in a safe environment
ALTER TABLE test_mytable ADD COLUMN new_column INT DEFAULT (existing_column + 1);
4. Populate Existing Rows with Computed Values Using DML

After adding the column, we need to populate it for all existing rows. This is where efficiency and performance become critical.

SQL
-- Update statement to backfill data
UPDATE mytable SET new_column = (existing_column + 1);
5. Create an Index on the New Column (Optional but Recommended)

If the new column will be frequently queried or used in joins, consider creating an index for performance optimization.

SQL
-- Create index on new column
CREATE INDEX idx_new_column ON mytable(new_column);
6. Ensure Repeatable and Forward-Only Migration

To ensure that running the migration script multiple times does not cause issues, we can wrap the DDL and DML operations in a transaction or use conditional logic to check if the column already exists.

SQL
-- Check if column exists before adding it
DO $$
BEGIN
   IF NOT EXISTS (SELECT 1 FROM information_schema.columns WHERE table_name = 'mytable' AND column_name = 'new_column') THEN
       ALTER TABLE mytable ADD COLUMN new_column INT DEFAULT (existing_column + 1);
   END IF;
END $$;

-- Backfill data only if the column does not already have values
DO $$
BEGIN
   IF NOT EXISTS (SELECT 1 FROM mytable WHERE new_column IS NULL) THEN
       UPDATE mytable SET new_column = (existing_column + 1);
   END IF;
END $$;

#### Closing Summary
and Common Mistakes/Follow-ups

By following the steps outlined, we ensure that the migration is repeatable and forward-only. The key points are:

Back up the database before making any changes.
Use DDL to add the new column with a computed default value.
Use an UPDATE statement to backfill existing rows efficiently.
Optionally create an index for performance optimization.

#### Common mistakes
include:

Failing to backup the database.
Not validating the script in a safe environment.
Ignoring potential performance issues during data population.

For follow-ups, consider discussing alternative methods like using a staging table or parallel processing if the dataset is extremely large and performance is critical.

**Comparison:**

| Step | Action |
| --- | --- |
| 1 | Back up the database |
| 2 | Add column with computed default value |
| 3 | Populate existing rows with computed values |
| 4 | Optionally create an index on new column |

**Code:**

```sql
-- DDL to add column with computed default
ALTER TABLE mytable ADD COLUMN new_column INT DEFAULT (existing_column + 1);

-- Update statement for backfilling data
UPDATE mytable SET new_column = (existing_column + 1);

-- Optional index creation
CREATE INDEX idx_new_column ON mytable(new_column);
```

**Example input / output:**

```text
-- Before migration
SELECT * FROM mytable;

-- After adding column with computed default and backfilling data
UPDATE mytable SET new_column = (existing_column + 1);

-- Final state after migration
SELECT * FROM mytable;
```

**Data:**

```json
{
    "mytable": {
        "columns": ["id", "name", "existing_column"],
        "data": [
            {"id": 1, "name": "row1", "existing_column": 5},
            {"id": 2, "name": "row2", "existing_column": 3}
        ]
    }
}
```

**Flow:**

1. Back up Database
2. Add Column
3. Validate Script
4. Populate Data
5. Check Index Needs
6. Create Index if Needed
7. Ensure Repeatable Migration

