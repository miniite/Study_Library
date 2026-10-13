---
type: "practice"
difficulty: "Deep"
---

# How do you add a NOT NULL constraint to an existing column that already has some NULLs, without a full-table exclusive lock?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Concurrent Index Builds](<../../../../01_Databases_and_SQL/Database_Operations/Migrations/Concurrent_Index_Builds/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of adding a NOT NULL constraint to an existing database column that contains NULL values without causing a full table exclusive lock, we need to adopt a strategic approach involving multiple steps and considerations. This is crucial in high-traffic environments where minimizing downtime and ensuring data integrity are paramount.

#### Triaging the Question

The primary goal here is to modify a column's schema while dealing with existing NULL values in an optimized manner. The interviewer is likely probing your knowledge of database management systems, transactional behavior, constraint handling, and best practices for minimizing performance impacts during schema changes. Key assumptions include:

You are working with a relational database system that supports transactional operations.
The database environment has a versioning strategy to manage migrations.

#### Solution
Step 1: Identify the Column and Check Existing Data

First, ensure you understand which column needs the NOT NULL constraint and verify its current state. For instance:

SQL
SELECT COUNT(*) FROM your_table WHERE column_name IS NULL;

This helps in assessing how many rows contain NULLs.

Step 2: Create an Index on the Column (Optional but Recommended)

Adding an index can improve query performance, especially if you plan to run further operations like updates. However, this step is optional:

SQL
CREATE INDEX idx_column_name ON your_table(column_name);
Step 3: Update Existing NULL Values

To handle rows with NULL values, you need to update them before adding the NOT NULL constraint. This can be done in a single transaction if the number of NULLs is manageable or in smaller batches for larger tables:

SQL
UPDATE your_table SET column_name = default_value WHERE column_name IS NULL;

Here, default_value should ideally match the expected type and content requirements of the column.

Step 4: Add the NOT NULL Constraint

Once all existing NULL values are handled, you can add the constraint. This step is critical as it makes the column immutable to future NULL assignments:

SQL
ALTER TABLE your_table ADD CONSTRAINT check_column_name_not_null CHECK (column_name IS NOT NULL);

#### Closing Summary
and Common Mistakes/Follow-Ups

This approach ensures that the migration process is both efficient and safe, minimizing downtime. However, common mistakes include not properly handling NULL values before adding constraints or failing to account for transactional impacts during large-scale changes.

#### Follow-Up
Questions:

How would you handle this in a distributed database system?
What if you have foreign key dependencies on that column?
Can you optimize the update process further?

**Comparison:**

```
Operation	SQL Statement
Check NULL count	SELECT COUNT(*) FROM your_table WHERE column_name IS NULL;
Create index	CREATE INDEX idx_column_name ON your_table(column_name);
Update NULL values	UPDATE your_table SET column_name = 'default_value' WHERE column_name IS NULL;
Add NOT NULL constraint	ALTER TABLE your_table ADD CONSTRAINT check_column_name_not_null CHECK (column_name IS NOT NULL);
```

**Flow:**

1. Yes
2. No
3. Check for NULL values
4. Are there many NULLs?
5. Update rows with defaultvalues
6. Add NOT NULL constraint

