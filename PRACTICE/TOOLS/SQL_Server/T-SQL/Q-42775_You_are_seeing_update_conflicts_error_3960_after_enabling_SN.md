---
type: "practice"
difficulty: "Deep"
---

# You are seeing update conflicts (error 3960) after enabling SNAPSHOT isolation. Explain why and how you would handle them in code.

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ MERGE Pitfalls and Error Handling](<../../../../TOOLS/SQL_Server/T-SQL/MERGE_Pitfalls_and_Error_Handling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When Snapshot Isolation is enabled, SQL Server uses a version store to maintain old versions of rows that are modified by transactions running under this isolation level. An update conflict occurs (error 3960) when two or more concurrent transactions attempt to modify the same row in such a way that one transaction's changes cannot be applied without conflicting with another’s.

To handle these conflicts, you need to understand why they occur and implement strategies to resolve them in your application logic. This involves recognizing the nature of the conflict and ensuring that your code can gracefully manage it or prevent it from happening in the first place.

Triaging the Problem

Question: Why do update conflicts (error 3960) happen after enabling SNAPSHOT isolation?

Concurrency: With Snapshot Isolation, transactions see a snapshot of the database at the start time of their transaction. If another transaction modifies data that is being read by an existing transaction, and both try to modify the same row, you can end up with conflicting updates.

Transaction Start Time: The key point is that the first transaction started its snapshot before the second one modified the row. When the first transaction tries to commit changes that were based on a now-obsolete version of the data, SQL Server detects this and throws an error 3960.

#### First-Principles Solution

Step-by-Step Solution:

Identify Conflicting Transactions: Use sys.dm_tran_locks or sys.dm_exec_requests to identify which transactions are conflicting with each other. You can also use SQL Server Profiler to trace the conflicting operations and understand their flow.

Transaction Design Review:

Ensure that your application logic respects transaction boundaries.
Validate whether your transactions are too long, causing them to span multiple business processes.
Check for any shared data that could be accessed by multiple transactions simultaneously.

Conflict Resolution Strategies:

Retry Logic: Implement retry logic in your application to handle transient conflicts. This can involve adding a delay and re-attempting the transaction until it succeeds.
Optimistic Concurrency Control: Use rowversion or timestamp columns to detect changes made by other transactions before you commit yours. If a conflict is detected, roll back and retry.

Code Example:

SQL
-- Example of using TRY...CATCH for handling conflicts in T-SQL

BEGIN TRY
    BEGIN TRANSACTION;

    -- Business logic here
    UPDATE Orders
    SET OrderStatus = 'Shipped'
    WHERE OrderID = 123 AND RowVersion = @RowVersion;

    COMMIT TRANSACTION;
END TRY
BEGIN CATCH
    IF ERROR_NUMBER() = 3960
    BEGIN
        PRINT 'Conflict detected. Retrying the transaction.';
        ROLLBACK TRANSACTION;
        -- Retry with updated version number
        UPDATE Orders
        SET OrderStatus = 'Shipped', RowVersion = @RowVersion + 1
        WHERE OrderID = 123 AND RowVersion = @RowVersion;

        BEGIN TRANSACTION;
        UPDATE Orders
        SET OrderStatus = 'Shipped'
        WHERE OrderID = 123 AND RowVersion = @RowVersion + 1;

        COMMIT TRANSACTION;
    END
    ELSE
    BEGIN
        PRINT 'Other error occurred: ' + ERROR_MESSAGE();
        ROLLBACK TRANSACTION;
    END
END CATCH;

Optimize Isolation Levels: Consider reducing the isolation level if possible, but be cautious as this might reduce concurrency.

Partition Data: If certain operations are frequent and conflict-prone, consider partitioning data to spread out contention.

Query Optimizations: Ensure that your queries are optimized for performance and minimize blocking by using proper indexing and query designs.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, the primary issue with update conflicts under Snapshot Isolation is due to concurrent modifications of rows within the same transactional context. To handle this in code, you should implement retry logic or other conflict resolution strategies. Always review your transaction boundaries and consider optimizations to reduce contention.

#### Common Mistakes

Ignoring transaction boundaries and allowing long-running transactions.
Not using TRY...CATCH blocks for proper error handling.
Failing to optimize queries and indexing, leading to higher contention.

For follow-ups, the interviewer might ask about specific database design choices or advanced concurrency control techniques. Be prepared to discuss how you would apply these concepts in a real-world scenario.

**Comparison:**

```
Database Server
Application
Database Server
Application
alt
[Conflict Detected]
Start Transaction
Update OrderStatus = 'Shipped'
RowVersion = 123
Commit Transaction
Error 3960
Retry with updated RowVersion
Success
```

**Code:**

```sql
BEGIN TRY
    BEGIN TRANSACTION;

    -- Business logic here
    UPDATE Orders
    SET OrderStatus = 'Shipped'
    WHERE OrderID = 123 AND RowVersion = @RowVersion;

    COMMIT TRANSACTION;
END TRY
BEGIN CATCH
    IF ERROR_NUMBER() = 3960
    BEGIN
        PRINT 'Conflict detected. Retrying the transaction.';
        ROLLBACK TRANSACTION;
        -- Retry with updated version number
        UPDATE Orders
        SET OrderStatus = 'Shipped', RowVersion = @RowVersion + 1
        WHERE OrderID = 123 AND RowVersion = @RowVersion;

        BEGIN TRANSACTION;
        UPDATE Orders
        SET OrderStatus = 'Shipped'
        WHERE OrderID = 123 AND RowVersion = @RowVersion + 1;

        COMMIT TRANSACTION;
    END
    ELSE
    BEGIN
        PRINT 'Other error occurred: ' + ERROR_MESSAGE();
        ROLLBACK TRANSACTION;
    END
END CATCH;
```

**Example input / output:**

```

Sample input:

SQL
Copy
-- Initialize the row version and start a transaction
UPDATE Orders
SET OrderStatus = 'Shipped', RowVersion = 123
WHERE OrderID = 123;

-- Start a new session and modify the same row
BEGIN TRANSACTION;
UPDATE Orders SET OrderStatus = 'Delivered' WHERE OrderID = 123 AND RowVersion = 124;
COMMIT TRANSACTION;

-- In the original transaction, update fails with error 3960
```

**Data:**

```sql
-- Sample schema for Orders table
CREATE TABLE Orders (
    OrderID INT PRIMARY KEY,
    OrderStatus NVARCHAR(50),
    RowVersion ROWVERSION NOT NULL
);
```

**Flow:**

1. Conflicting Update
2. Yes
3. No
4. Start Transaction
5. Check RowVersion
6. Rollback
7. Retry with Updated Version?
8. Commit Transaction
9. End Transaction
10. Conflict Not Detected
11. Update Succeeded
12. Commit Transaction

