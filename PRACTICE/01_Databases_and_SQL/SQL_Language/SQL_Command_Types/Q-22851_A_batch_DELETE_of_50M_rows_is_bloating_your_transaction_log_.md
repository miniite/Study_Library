---
type: "practice"
difficulty: "Intermediate"
---

# A batch DELETE of 50M rows is bloating your transaction log and holding locks. How would you restructure it into safe chunked deletes?

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ DELETE vs TRUNCATE](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Command_Types/DELETE_vs_TRUNCATE.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with a large DELETE operation (in this case, 50 million rows) that is causing excessive transaction logging and locking issues, restructuring the operation into smaller, incremental deletions can help mitigate these problems. The key concepts here are understanding DDL/DML/DCL/TCL in the context of SQL transactions and how they interact with the database.

#### Triage
What's Being Probed

The interviewer is likely probing your understanding of transaction management, logging mechanisms, concurrency control, and best practices for large-scale data manipulations. They will be interested in how you handle performance, locking, and log management in a production environment.

#### Assumptions
We are working with a relational database system (e.g., PostgreSQL, MySQL, SQL Server).
The table has significant transaction logging enabled.
There is no immediate need for the deleted data to be immediately available.
Step-by-Step Solution
1. Identify Key Constraints and Dependencies

First, identify any constraints or dependencies that might interfere with a safe chunked delete:

SQL
SELECT CONSTRAINT_NAME
FROM INFORMATION_SCHEMA.TABLE_CONSTRAINTS
WHERE TABLE_NAME = 'your_table';
2. Design the Chunked Delete Strategy

To avoid bloating the transaction log and ensuring database performance, we can use BEGIN TRANSACTION and COMMIT to manage smaller batches of deletions:

SQL
-- Example for PostgreSQL
DO $$
DECLARE
    chunk_size INT := 10000;
    start_row INT := 0;
    end_row INT := start_row + chunk_size - 1;
BEGIN
    WHILE True LOOP
        -- Find the rows to delete in this batch
        WITH row_to_delete AS (
            SELECT id
            FROM your_table
            WHERE condition_for_deletion AND id > start_row LIMIT chunk_size
        )
        DELETE FROM your_table
        WHERE id IN (SELECT id FROM row_to_delete);

        -- Commit the transaction for this batch
        COMMIT;

        -- Prepare for the next batch
        GET DIAGNOSTICS row_count = ROW_COUNT;
        IF row_count < chunk_size THEN
            EXIT;  -- Stop if we've reached the end of deletions
        END IF;

        start_row := end_row + 1;
        end_row := start_row + chunk_size - 1;
    END LOOP;
END $$;
3. Monitor Performance and Locks

During execution, monitor the database performance metrics to ensure that the transaction log is not excessively bloated and that locks are managed properly:

SQL
-- Check lock status
SELECT * FROM pg_locks WHERE relation = 'your_table'::regclass;

-- Monitor transaction log usage
SELECT * FROM pg_stat_wal;

#### Closing Summary
and Common Mistakes
Common Mistake 1: Not committing after each chunk can lead to excessive log space consumption.
Common Mistake 2: Overlooking index dependencies or foreign key constraints which might cause locking issues.
Best Practice: Use WITH clauses for subqueries to improve readability and performance.

#### Example
Input & Output
SQL
-- Initial setup (for testing purposes)
CREATE TABLE your_table (
    id SERIAL PRIMARY KEY,
    data TEXT NOT NULL
);

-- Insert sample data
INSERT INTO your_table (data) SELECT 'data' FROM generate_series(1, 50000000);

#### Follow-Ups
How would you handle cases where the chunk size needs to be adjusted?
What are the implications of this approach on read operations during deletion?

**Code:**

```sql
DO $$
DECLARE
    chunk_size INT := 10000;
    start_row INT := 0;
    end_row INT := start_row + chunk_size - 1;
BEGIN
    WHILE True LOOP
        -- Find the rows to delete in this batch
        WITH row_to_delete AS (
            SELECT id
            FROM your_table
            WHERE condition_for_deletion AND id > start_row LIMIT chunk_size
        )
        DELETE FROM your_table
        WHERE id IN (SELECT id FROM row_to_delete);

        -- Commit the transaction for this batch
        COMMIT;

        -- Prepare for the next batch
        GET DIAGNOSTICS row_count = ROW_COUNT;
        IF row_count < chunk_size THEN
            EXIT;  -- Stop if we've reached the end of deletions
        END IF;

        start_row := end_row + 1;
        end_row := start_row + chunk_size - 1;
    END LOOP;
END $$;
```

**Example input / output:**

```text
-- Initial setup (for testing purposes)
CREATE TABLE your_table (
    id SERIAL PRIMARY KEY,
    data TEXT NOT NULL
);

-- Insert sample data
INSERT INTO your_table (data) SELECT 'data' FROM generate_series(1, 50000000);
```

**Data:**

```sql
CREATE TABLE your_table (
    id SERIAL PRIMARY KEY,
    data TEXT NOT NULL
);

INSERT INTO your_table (data) SELECT 'data' FROM generate_series(1, 50000000);
```

