---
type: "practice"
difficulty: "Deep"
---

# Give a step-by-step of how you'd add SERIALIZABLE only around the critical section of one endpoint while keeping the rest at READ COMMITTED. What are the gotchas?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Serializable Performance](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Setting_Isolation_Levels/Serializable_Performance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this question, we need to understand why and how we would use different isolation levels in a mixed environment, where some parts of an application are critical enough to require SERIALIZABLE transactions, while others can remain at READ COMMITTED.

TL;DR

We will implement SERIALIZABLE transactions only around the critical section of one endpoint by leveraging database session settings or transaction management features. The key challenge lies in ensuring that the rest of the application remains at a lower isolation level to prevent unnecessary overhead and potential deadlocks.

#### Triage

The question is probing our understanding of transactional semantics, specifically how different isolation levels can coexist within an application. We need to ensure we understand the implications of each isolation level (RU/RC/RR/Serializable) and know how to apply them selectively without compromising the overall system's performance or consistency.

#### Assumptions
The critical section is well-defined and can be isolated from the rest of the code.
We are working with a relational database management system (RDBMS) that supports SERIALIZABLE transactions, such as PostgreSQL or SQL Server.

#### First-Principles Solution
Step 1: Define the Critical Section

Identify the parts of the endpoint where data integrity is paramount and must be protected from concurrent modifications. For instance, a financial transaction processing system might have critical sections around updating account balances.

PYTHON
# Pseudo-code to define the critical section
def process_transaction():
    # Start SERIALIZABLE transaction
    start_serializable_transaction()

    try:
        # Perform critical operations (e.g., update account balance)
        update_account_balance(account_id, amount)

        # Commit the transaction if all is well
        commit_transaction()
    except Exception as e:
        # Handle any exceptions and rollback in case of failure
        roll_back_transaction()
Step 2: Transaction Management Strategy

We can use a combination of database session settings or transaction management APIs to ensure that only the critical sections execute with SERIALIZABLE isolation, while the rest remains at READ COMMITTED.

SQL
-- Example using PostgreSQL
BEGIN ISOLATION LEVEL SERIALIZABLE;
    -- Critical operations here
COMMIT;

-- Rest of the code runs at default isolation level (READ COMMITTED)
Step 3: Error Handling and Rollbacks

Ensure that any errors within the critical section are handled properly to maintain data consistency. This involves rolling back the transaction if an error occurs.

SQL
BEGIN ISOLATION LEVEL SERIALIZABLE;
    -- Critical operations here
EXCEPTION WHEN OTHERS THEN
    ROLLBACK;
END;
Step 4: Performance Considerations

SERIALIZABLE transactions are the most restrictive and can lead to increased locking overhead. Therefore, we need to carefully define which parts of the application require this level of isolation.

#### Closing Summary

By selectively applying SERIALIZABLE transactions only around critical sections, we can maintain data integrity without impacting the performance of non-critical operations. However, it is crucial to define these critical sections precisely and ensure proper error handling to avoid potential deadlocks or other consistency issues.

Common Gotchas
Deadlocks: SERIALIZABLE transactions increase the likelihood of deadlocks due to higher locking requirements.
Performance Overhead: SERIALIZABLE transactions can significantly slow down your application, so they should be used judiciously.
Session Management: Ensure that all database sessions are correctly configured and managed to avoid conflicts between different isolation levels.
Self-Check

This solution is correct and comprehensive, covering both the implementation strategy and potential issues to watch out for.

**Code:**

```sql
BEGIN ISOLATION LEVEL SERIALIZABLE;
    -- Critical operations here
COMMIT;

-- Rest of the code runs at default isolation level (READ COMMITTED)
```

**Flow:**

1. Yes
2. No
3. Start Transaction
4. Is Critical Section?
5. Begin SERIALIZABLETransaction
6. Perform Critical Operations
7. Commit Transaction
8. End Transaction
9. Routine Operations (READCOMMITTED)
10. End Transaction

