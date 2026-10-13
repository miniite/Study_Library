---
type: "practice"
difficulty: "Deep"
---

# Explain how retry logic for deadlock victims (error 1205) should be implemented at the application or procedure level, and what makes an operation safe to retry.

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Read Committed Locking Anomalies](<../../../../TOOLS/SQL_Server/SQL_Server_Platform/Read_Committed_Locking_Anomalies.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When implementing retry logic for deadlock victims in SQL Server, it's crucial to understand that deadlocks occur when two or more transactions are waiting for each other to release resources they hold, creating a circular dependency and causing both operations to hang indefinitely. Deadlock detection is automatic in SQL Server through the SPID (Session Process ID) and DBCC INPUTBUFFER, but handling these deadlocks requires custom logic at the application or stored procedure level.

To safely implement retry logic for deadlock victims, we need to ensure that we can accurately identify when a transaction has been terminated due to a deadlock, determine which parts of our operation are safe to reattempt, and handle the retries without introducing further deadlocks. The key steps involve:

Detecting Deadlock: Identifying the error 1205 (deadlock detected) in SQL Server.
Determining Safe Operations for Retrying: Ensuring that only operations that do not hold locks are retried, to avoid re-entering a deadlock cycle.

#### Triaging the Question

The core of this question revolves around understanding:

The nature and causes of deadlocks in SQL Server.
How to identify when a transaction has been terminated due to a deadlock.
Strategies for safely retrying operations without exacerbating the deadlock situation.

We will assume that the application is using T-SQL stored procedures or transactions. Common assumptions include the presence of error handling mechanisms, logging capabilities, and sufficient knowledge of the database schema.

#### First Principles Solution

Detecting Deadlock:

SQL Server raises a specific error 1205 when a deadlock occurs.
To handle this, you need to catch this error in your application or stored procedure logic using TRY...CATCH blocks.

Identifying Safe Operations for Retrying:

Once a deadlock is detected, the transaction that was terminated must be retried only if its operations do not hold any locks.
This can be achieved by:
Ensuring that each operation in your transaction is lightweight and does not acquire long-term locks.
Designing your transactions to break into smaller units of work where individual steps are safe to retry.
Implementing Retry Logic

Here’s a step-by-step implementation using T-SQL:

SQL
BEGIN TRY
    -- Perform operations that may result in a deadlock
    BEGIN TRANSACTION;

    INSERT INTO Employees (FirstName, LastName) VALUES ('John', 'Doe');
    UPDATE Departments SET ManagerID = 123 WHERE DepartmentID = 456;

    COMMIT TRANSACTION;
END TRY
BEGIN CATCH
    IF ERROR_NUMBER() = 1205 -- Check for deadlock error
    BEGIN
        -- Safe operations to retry: INSERT, no SELECTs that might acquire locks
        ROLLBACK TRANSACTION;

        -- Retry only the last operation that is safe to re-attempt
        BEGIN TRANSACTION;

        INSERT INTO Employees (FirstName, LastName) VALUES ('Jane', 'Doe'); -- Attempt this again

        IF @@TRANCOUNT > 0
            COMMIT TRANSACTION; -- Commit if no more lock conflicts
    END
END CATCH;

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, implementing retry logic for deadlock victims involves:

Detecting deadlocks using error handling.
Ensuring only safe operations are retried to avoid re-creating the deadlock.
Breaking down transactions into smaller, non-blocking units.

#### Common mistakes
include retrying all operations in a transaction without checking if they hold locks, leading to potential new deadlocks or resource starvation. Additionally, improper transaction management can lead to incomplete rollbacks and data inconsistencies.

#### Follow-ups
could involve discussing how to optimize the application for better concurrency, such as using row-level locking hints (WITH (ROWLOCK)), or implementing distributed transactions with a retry policy that includes waiting periods between retries to avoid exacerbating the deadlock situation.

**Code:**

```sql
BEGIN TRY
    -- Perform operations that may result in a deadlock
    BEGIN TRANSACTION;

    INSERT INTO Employees (FirstName, LastName) VALUES ('John', 'Doe');
    UPDATE Departments SET ManagerID = 123 WHERE DepartmentID = 456;

    COMMIT TRANSACTION;
END TRY
BEGIN CATCH
    IF ERROR_NUMBER() = 1205 -- Check for deadlock error
    BEGIN
        -- Safe operations to retry: INSERT, no SELECTs that might acquire locks
        ROLLBACK TRANSACTION;

        -- Retry only the last operation that is safe to re-attempt
        BEGIN TRANSACTION;

        INSERT INTO Employees (FirstName, LastName) VALUES ('Jane', 'Doe'); -- Attempt this again

        IF @@TRANCOUNT > 0
            COMMIT TRANSACTION; -- Commit if no more lock conflicts
    END
END CATCH;
```

