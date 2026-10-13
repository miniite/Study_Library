---
type: "practice"
difficulty: "Expert"
---

# Describe an expand-contract migration to rename a column without breaking a live application reading and writing the table.

**Difficulty:** Expert · **Levels:** SDE-1, SDE-1/2 · [↑ Temp Tables vs Table Variables](<../../../../TOOLS/SQL_Server/T-SQL/Temp_Tables_vs_Table_Variables.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the requirement of renaming a column in a T-SQL database schema without causing disruptions to a live application, we need to carefully plan an expand-contract migration strategy. This involves creating a new column, populating it with the data from the old column, dropping the old column, and then renaming the new column to replace the old one.

Firstly, let's break down what the question is really asking: how to perform a zero-downtime schema change in a live application that reads and writes a table. The key here is to ensure that there are no disruptions during the migration process. We will be using T-SQL commands within an SQL Server environment.

We will assume:

The live application performs read-write operations on the table.
The old column name is oldColumn.
The new column name will be newColumn.
Step-by-Step Solution

Add a New Column:

Create the new column with the desired data type and constraints, ensuring it has the same or compatible data type as the old column.

Populate the New Column:

Write an SQL script to populate the new column with the data from the existing oldColumn. This can be done in batches for large tables to avoid locking issues.

Drop Constraints and Indexes (if necessary):

If there are any constraints or indexes on the old column, drop them before proceeding.

Update the New Column:

Use a trigger or scheduled batch processing to update the new column with the data from oldColumn during read operations. Alternatively, you can use a stored procedure that updates both columns in bulk when writes occur.

Drop the Old Column:

Once the new column is populated and the application is handling reads and writes using the new column, drop the old column.

Rename the New Column to Replace the Old One:

Finally, rename the new column to oldColumn to complete the migration.
Full Migration Plan
SQL
-- Step 1: Add a new column with the same data type as the old column.
ALTER TABLE YourTable ADD newColumn DataType;

-- Step 2: Populate the new column during read operations.
CREATE TRIGGER trg_PopulateNewColumn ON YourTable FOR SELECT AS
BEGIN
    UPDATE YourTable
    SET newColumn = (SELECT oldColumn FROM inserted);
END;

-- Step 3: Drop constraints and indexes on the old column if necessary.
ALTER TABLE YourTable DROP CONSTRAINT constraintName;
DROP INDEX indexName ON YourTable;

-- Step 4: Update the new column during write operations using a stored procedure.
CREATE PROCEDURE sp_UpdateOldAndNewColumns
    @oldColumn DataType,
    -- other parameters as needed
AS
BEGIN
    UPDATE YourTable
    SET oldColumn = @newColumn, newColumn = @newColumn;
END;

-- Step 5: Drop the old column once reads and writes are using the new column.
ALTER TABLE YourTable DROP COLUMN oldColumn;

-- Step 6: Rename the new column to replace the old one.
EXEC sp_rename 'YourTable.newColumn', 'oldColumn', 'COLUMN';

#### Closing Summary

This expand-contract migration ensures zero downtime by temporarily adding a new column, populating it with existing data, and then replacing the old column. The key steps involve creating the new column, handling reads and writes to update the new column, dropping the old column, and renaming the new one.

#### Common Mistakes/Follow-Ups
Not Handling Concurrency: Ensure that your solution handles concurrent read/write operations without deadlocks or locking issues.
Data Integrity: Verify data integrity during the migration process by running checks before and after each step.
Performance Impact: Monitor performance impact, especially with large tables, to ensure that the process does not degrade application responsiveness.

