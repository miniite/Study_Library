---
type: "practice"
difficulty: "Deep"
---

# What is an autonomous transaction in Oracle PL/SQL and what is a legitimate use case such as logging inside a failing transaction?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Oracle Isolation and Read Consistency](<../../../../TOOLS/Oracle/Oracle_Platform/Oracle_Isolation_and_Read_Consistency.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

An autonomous transaction in Oracle PL/SQL is a special kind of transaction that runs independently from the main transaction. This means it can commit or rollback its own changes, which can be crucial when you need to maintain data integrity even if a primary operation fails. A legitimate use case for an autonomous transaction is logging events inside a failing transaction without disrupting the main transaction’s flow.

#### Triage
The question asks about autonomous transactions and their application in logging within a failing transaction. We need to clarify that autonomous transactions are separate units of work, allowing us to handle specific operations independently from the main transaction context. This ensures that critical tasks like logging can continue even if the primary operation encounters issues.

Step-by-Step Solution:

Understanding Autonomous Transactions: An autonomous transaction is started with a BEGIN statement and committed or rolled back using COMMIT WORK; or ROLLBACK WORK;. These transactions are independent of the main transaction, allowing them to commit or rollback their changes without affecting the outer transaction.
Use Case: Logging in Failing Transactions: When you have a primary operation that might fail due to various reasons (e.g., constraints violations), logging these events is essential for debugging and maintaining logs. However, if the logging process itself fails, it should not roll back the entire transaction.

#### Example Scenario
Consider an application where a user updates their account details. If the update operation fails due to, say, violating a unique constraint, you want to log this event so that support teams can investigate later. Here’s how you might implement autonomous transactions for logging:

SQL
DECLARE
  -- Main transaction
  l_result INTEGER;
BEGIN
  BEGIN
    -- Autonmous transaction starts here
    EXECUTE IMMEDIATE 'ALTER SESSION ENABLE DEBURG HINT (SET SQLFORMAT TEXT)';

    -- Main update operation
    l_result := UPDATE accounts SET email = 'newemail@example.com' WHERE user_id = 123;

    -- Commit main transaction only if no errors occurred
    IF l_result > 0 THEN
      COMMIT;
    ELSE
      ROLLBACK WORK; -- Fail the main transaction

      -- Log the failure in an autonomous transaction
      BEGIN
        EXECUTE IMMEDIATE 'INSERT INTO account_log (user_id, action, log_time) VALUES (123, ''UPDATE_FAILED'', SYSDATE)';
      EXCEPTION WHEN OTHERS THEN
        ROLLBACK WORK; -- Ensure logging does not fail main operation
      END;
    END IF;
  EXCEPTION WHEN OTHERS THEN
    ROLLBACK WORK; -- Rollback main transaction on any error
  END;
END;

#### Closing Summary
Autonomous transactions are a powerful feature in PL/SQL that enable you to handle independent units of work. They allow critical operations like logging to continue even if the primary operation fails, ensuring data integrity and providing valuable debug information.

#### Common Mistakes/Follow-ups

Not handling exceptions properly within autonomous transactions can lead to incomplete or incorrect logs.
Failing to distinguish between committing an autonomous transaction and rolling it back can result in unintended changes being logged.

**Code:**

```sql
DECLARE
  l_result INTEGER;
BEGIN
  BEGIN
    EXECUTE IMMEDIATE 'ALTER SESSION ENABLE DEBURG HINT (SET SQLFORMAT TEXT)';

    -- Main update operation
    l_result := UPDATE accounts SET email = 'newemail@example.com' WHERE user_id = 123;

    IF l_result > 0 THEN
      COMMIT;
    ELSE
      ROLLBACK WORK; -- Fail the main transaction

      BEGIN
        EXECUTE IMMEDIATE 'INSERT INTO account_log (user_id, action, log_time) VALUES (123, ''UPDATE_FAILED'', SYSDATE)';
      EXCEPTION WHEN OTHERS THEN
        ROLLBACK WORK; -- Ensure logging does not fail main operation
      END;
    END IF;
  EXCEPTION WHEN OTHERS THEN
    ROLLBACK WORK; -- Rollback main transaction on any error
  END;
END;
```

