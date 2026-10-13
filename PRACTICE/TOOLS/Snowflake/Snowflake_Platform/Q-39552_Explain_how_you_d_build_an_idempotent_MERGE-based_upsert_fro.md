---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd build an idempotent MERGE-based upsert from a stream so that re-running a failed task never double-applies rows.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Snowflake Loading and Pipelines](<../../../../TOOLS/Snowflake/Snowflake_Platform/Snowflake_Loading_and_Pipelines.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To build an idempotent MERGE-based upsert from a stream in Snowflake, we need to ensure that re-running a failed task doesn't result in duplicated insertions or updates. The core idea is to leverage primary keys and conditional logic within the MERGE statement to handle concurrent writes gracefully. This solution involves using unique identifiers, proper conflict resolution, and potentially storing metadata about the last successful execution.

First, let's break down what we are solving: We need a mechanism that ensures each row in our target table is updated or inserted only once, even if the same stream events are re-applied due to task failures. This requires handling primary keys effectively and managing concurrent transactions properly.

To achieve this, follow these steps:

Define the Primary Key: Ensure your source data has a unique identifier (e.g., id) that Snowflake can use to uniquely identify each row.
Design the MERGE Statement: Construct the MERGE statement in such a way that it only updates or inserts rows conditionally, based on these primary keys.
Use Metadata Table: Maintain a metadata table to track which rows have been processed successfully.

Now let's dive into the detailed solution:

Setup Primary Keys and Metadata Table:
In your target Snowflake table (target_table), ensure you have a unique identifier column (e.g., id) that uniquely identifies each row.
Create a metadata table to track which rows have been processed successfully.
SQL
CREATE TABLE IF NOT EXISTS metadata (
    id INTEGER,
    last_processed TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (id)
);
Define the MERGE Statement: The core of our solution is crafting the MERGE statement correctly. This involves checking if a row already exists based on its unique identifier and updating or inserting it conditionally.
SQL
MERGE INTO target_table AS tgt
USING stream_data AS src
ON tgt.id = src.id  -- Match rows based on the primary key

-- Conditional insert/update logic
WHEN NOT MATCHED THEN
    INSERT (id, column1, column2) VALUES (src.id, src.column1, src.column2)
WHEN MATCHED THEN
    UPDATE SET
        column1 = COALESCE(src.column1, tgt.column1),
        column2 = COALESCE(src.column2, tgt.column2);
Use Metadata for Idempotence: To ensure that re-running the task doesn’t duplicate work, we can use a MERGE statement with additional checks using the metadata table.
SQL
WITH updated_rows AS (
    MERGE INTO target_table AS tgt
    USING stream_data AS src
    ON tgt.id = src.id  -- Match rows based on the primary key

    WHEN NOT MATCHED THEN
        INSERT (id, column1, column2) VALUES (src.id, src.column1, src.column2)

    WHEN MATCHED THEN
        UPDATE SET
            column1 = COALESCE(src.column1, tgt.column1),
            column2 = COALESCE(src.column2, tgt.column1),

    RETURNING 1
)

INSERT INTO metadata (id)
SELECT id FROM updated_rows;

Handling Concurrent Writes: Ensure that concurrent writes are managed properly by using transaction isolation levels and proper locking mechanisms.

Monitoring and Logging: Implement logging to track which rows have been processed successfully, helping with debugging and performance monitoring.

**Comparison:**

|   | ID | Column1 | Column2 |
|---|----|---------|---------|
| 0 | 1  | valueA  | valueB  |

**Code:**

```sql
CREATE TABLE IF NOT EXISTS metadata (
    id INTEGER,
    last_processed TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (id)
);

WITH updated_rows AS (
    MERGE INTO target_table AS tgt
    USING stream_data AS src
    ON tgt.id = src.id  -- Match rows based on the primary key

    WHEN NOT MATCHED THEN
        INSERT (id, column1, column2) VALUES (src.id, src.column1, src.column2)

    WHEN MATCHED THEN
        UPDATE SET
            column1 = COALESCE(src.column1, tgt.column1),
            column2 = COALESCE(src.column2, tgt.column1),

    RETURNING 1
)

INSERT INTO metadata (id)
SELECT id FROM updated_rows;
```

**Example input / output:**

```text
WITH updated_rows AS (
    MERGE INTO target_table AS tgt
    USING stream_data AS src
    ON tgt.id = src.id  -- Match rows based on the primary key

    WHEN NOT MATCHED THEN
        INSERT (id, column1, column2) VALUES (src.id, src.column1, src.column2)

    WHEN MATCHED THEN
        UPDATE SET
            column1 = COALESCE(src.column1, tgt.column1),
            column2 = COALESCE(src.column2, tgt.column1),

    RETURNING 1
)

INSERT INTO metadata (id)
SELECT id FROM updated_rows;
```

**Data:**

```sql
-- Sample data for stream_data table
CREATE TABLE IF NOT EXISTS stream_data (
    id INTEGER,
    column1 VARCHAR,
    column2 VARCHAR
);

-- Sample insert into stream_data
INSERT INTO stream_data (id, column1, column2) VALUES (1, 'valueA', 'valueB');
```

**Flow:**

1. Yes
2. No
3. Source Data Stream
4. Is Metadata Table Updated?
5. Mark Row as Processed inMetadata
6. Merge into Target Table
7. Insert into Metadata for NewRows

