---
type: "practice"
difficulty: "Deep"
---

# You ran an ALTER TABLE ADD COLUMN on a 200M-row table in production and it locked writes for minutes. How would you make that change online with minimal disruption?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Online Schema Changes](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Online_Schema_Changes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When faced with the challenge of adding a column to a large table in production without locking writes, we need to consider using an "online DDL" approach. This involves making the schema change in a way that minimizes downtime and disruption. The core idea is to use temporary tables or partitioning techniques to create the new schema incrementally.

First, let's break down what we are trying to achieve:

Add the new column without locking writes.
Populate the new column with existing data (if necessary).
Migrate rows in batches if needed.
Update indexes and constraints, ensuring consistency.
Drop old schema once the new one is stable.
Step-by-Step Solution

Create a Temporary Table: We can create a temporary table with all the existing columns from the original table plus the new column. This allows us to incrementally populate the data in parallel without blocking writes on the main table.

SQL
CREATE TABLE my_table_temp AS SELECT * FROM my_table;
ALTER TABLE my_table_temp ADD COLUMN new_column_name datatype;

Populate the New Column: Use a MERGE statement or an UPDATE statement to populate the new column with existing data if necessary.

SQL
MERGE INTO my_table_temp AS target
USING (SELECT id, existing_col1 FROM my_table) AS source
ON (target.id = source.id)
WHEN MATCHED THEN UPDATE SET target.new_column_name = source.existing_col1;

Batches and Incremental Updates: To avoid overwhelming the database during population, use batches to add rows incrementally.

SQL
-- Example batch update
BEGIN TRANSACTION;

INSERT INTO my_table_temp (id, existing_col1, new_column_name)
SELECT id, existing_col1, CASE WHEN row_number() <= 50000 THEN 'default_value' ELSE NULL END
FROM (SELECT ROW_NUMBER() OVER (ORDER BY id) AS rn, * FROM my_table);

COMMIT;

Indexes and Constraints: Ensure that any indexes or constraints are created after the data migration to avoid locking issues.

SQL
CREATE INDEX idx_new_column ON my_table_temp(new_column_name);
ALTER TABLE my_table_temp ADD CONSTRAINT constraint_name CHECK (new_column_name IS NOT NULL);

Switch Tables Using Partitioning: Once the new column is populated, you can use a technique like partition exchange to swap the old and new tables.

SQL
-- Example using partition exchange
ALTER TABLE my_table EXCHANGE PARTITION FOR ('current_partition_key') WITH TABLE my_table_temp;

Final Cleanup: Drop the temporary table once everything is stable.

SQL
DROP TABLE my_table_temp;

#### Closing Summary
and Common Mistakes

This approach minimizes downtime by performing the schema changes in a way that does not block writes on the main table. However, there are common pitfalls:

Overlooking concurrency issues: Ensure that no other processes access the data during migration.
Insufficient testing: Always test your script in a staging environment to ensure it works as expected before running in production.
Resource Management: Be mindful of database resources like memory and I/O when performing large updates.

By following these steps, you can achieve an online DDL operation that minimizes disruption. Remember, the key is to break down the process into manageable parts and test thoroughly.

**Code:**

```sql
CREATE TABLE my_table_temp AS SELECT * FROM my_table;
ALTER TABLE my_table_temp ADD COLUMN new_column_name datatype;

MERGE INTO my_table_temp AS target
USING (SELECT id, existing_col1 FROM my_table) AS source
ON (target.id = source.id)
WHEN MATCHED THEN UPDATE SET target.new_column_name = source.existing_col1;

BEGIN TRANSACTION;
INSERT INTO my_table_temp (id, existing_col1, new_column_name)
SELECT id, existing_col1, CASE WHEN row_number() <= 50000 THEN 'default_value' ELSE NULL END
FROM (SELECT ROW_NUMBER() OVER (ORDER BY id) AS rn, * FROM my_table);
COMMIT;

CREATE INDEX idx_new_column ON my_table_temp(new_column_name);
ALTER TABLE my_table_temp ADD CONSTRAINT constraint_name CHECK (new_column_name IS NOT NULL);

ALTER TABLE my_table EXCHANGE PARTITION FOR ('current_partition_key') WITH TABLE my_table_temp;
DROP TABLE my_table_temp;
```

**Flow:**

1. Rendering diagram…

