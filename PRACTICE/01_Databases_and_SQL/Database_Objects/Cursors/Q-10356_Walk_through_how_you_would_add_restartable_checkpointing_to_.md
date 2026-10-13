---
type: "practice"
difficulty: "Deep"
---

# Walk through how you would add restartable checkpointing to a cursor-driven ETL so a failure at row 800k does not reprocess from zero.

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Ref Cursor Consistency and Limits](<../../../../01_Databases_and_SQL/Database_Objects/Cursors/Ref_Cursor_Consistency_and_Limits.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the requirement of adding restartable checkpointing to a cursor-driven ETL process, we need to ensure that in case of a failure, the process can resume exactly where it left off without having to reprocess from the beginning. This involves implementing a mechanism for tracking the current position and ensuring that only subsequent rows are processed after a failure.

The key idea is to leverage database transactions and metadata storage to maintain the state information required for restarting the ETL process at any point. Let's break down the solution step by step:

Identify the Current Position: We need to store the current position of the cursor in a way that it can be easily accessed after a failure.
Transaction Management: Use database transactions to ensure data integrity during state updates and ETL processing.
Metadata Storage: Store metadata about the last processed row or any other relevant state information.

#### Triage
What's Being Probed

The interviewer is probing your understanding of:

Cursor-based data processing.
Transaction management in databases.
Metadata storage strategies.
Error handling and recovery mechanisms.

#### Trade-offs
between implementation complexity, performance, and reliability.

#### Assumptions
The ETL process is driven by a cursor that reads rows from a source database and writes them to a target database or a staging area.
The system can handle the overhead of transactions and metadata storage without significant impact on performance.
There are no constraints on the amount of metadata stored for each transaction.

#### First-Principles Solution
Transaction Management:
Each batch of rows processed by the cursor should be wrapped in a database transaction to ensure atomicity.
Metadata Storage:
Maintain a table or a separate storage mechanism (e.g., a column in the target table, an additional metadata table) to store the current position.
Checkpointing Logic:
Before starting a new batch of rows, check if there is a previous checkpoint stored.
If a checkpoint exists, start processing from that point onwards.
Error Handling and Recovery:
In case of a failure during processing, roll back the transaction to maintain consistency.
Write the current position as a new checkpoint before exiting.

#### Implementation
Steps:

Identify Current Position:

Maintain a column or table entry that stores the last processed row ID.

Transaction Wrapper:

SQL
-- Start a transaction for each batch of rows
BEGIN TRANSACTION;

-- Process rows from the current position to the next checkpoint
INSERT INTO target_table (SELECT * FROM source_table WHERE id > @lastProcessedID);

-- Update the last processed row ID as a new checkpoint
UPDATE metadata_table SET lastProcessedID = MAX(source_table.id) + 1;

Error Handling:

Use try-catch blocks to handle exceptions and rollback transactions.
SQL
BEGIN TRY
    -- Transaction logic here
END TRY
BEGIN CATCH
    ROLLBACK TRANSACTION;
    -- Log the error and update metadata accordingly
END CATCH

#### Closing Summary

By implementing restartable checkpointing, we ensure that in case of a failure, the ETL process can resume exactly where it left off. This approach leverages database transactions for atomicity and metadata storage to maintain state information.

#### Common Mistakes/Follow-Ups
Not Handling Transactions Properly: Failing to wrap each batch in a transaction could lead to data inconsistencies.
Ignoring Metadata Storage: Not maintaining proper metadata can result in incomplete or incorrect restart points.
Performance Overhead: The overhead of transactions and additional metadata storage should be considered, especially for large datasets.

**Example input / output:**

```text
Input:
- Source table with IDs from 1 to 1,000,000.
- Last processed row ID stored in `metadata_table` as 800,000.

Output:
- After processing rows 800,001 to 1,000,000 and committing the transaction, `lastProcessedID` in `metadata_table` should be updated to 1,000,001.
```

