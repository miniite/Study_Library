---
type: "practice"
difficulty: "Deep"
---

# What is a deadlock in MySQL and how does InnoDB detect and resolve one?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Deadlocks in MySQL](<../../../../TOOLS/MySQL/MySQL_Platform/Deadlocks_in_MySQL.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Deadlocks in MySQL are a concurrency issue where two or more threads simultaneously block each other, waiting for resources held by the others. InnoDB detects deadlocks through a cycle detection algorithm and resolves them by rolling back one of the transactions involved. Let's break down the key concepts and steps from first principles.

First, let's define what a deadlock is: It occurs when two or more threads are blocked forever because each is waiting for the other to release a resource. In MySQL with InnoDB, this typically happens when multiple transactions acquire locks on different resources in a circular wait pattern.

Now, let’s understand how InnoDB detects and resolves deadlocks:

Transaction Locks: Each transaction acquires locks on data as needed.
Lock Order Management (LKM): InnoDB uses LKM to ensure that the order of lock acquisition is consistent across transactions. This helps prevent circular waits but doesn't guarantee a deadlock will never occur.
Deadlock Detection

InnoDB periodically checks for deadlocks using the following steps:

Cycle Detection: At commit time or on explicit SELECT ... FOR UPDATE, InnoDB scans all current transactions to detect cycles in the lock graph.
Marking Transactions: When a cycle is detected, InnoDB marks these transactions as "involvement set" and tracks their status.
Rollback: The transaction with the smallest duration (i.e., least committed work) or with a smaller autocommit value is chosen to be rolled back. This minimizes the impact of the rollback on ongoing operations.

#### Example Scenario

Let's consider two transactions, T1 and T2:

T1: Locks A, then waits for B.
T2: Locks B, then waits for A.

This creates a deadlock where neither transaction can proceed without releasing locks held by the other. InnoDB will detect this cycle and resolve it by rolling back one of the transactions.

Code Flow

Here’s a simplified pseudocode outline for how InnoDB might handle this:

SQL
-- Pseudocode for deadlock detection and resolution
BEGIN TRANSACTION;

FOR each transaction T DO
    IF T is in involvement set THEN
        -- Check if cycle exists involving T and others
        IF cycle detected THEN
            SELECT smallest_transaction TO rollback;
            ROLLBACK smallest_transaction;
            BREAK;
        ENDIF;
    ENDIF;
END FOR;

COMMIT;

#### Closing Summary

In summary, deadlocks are critical to handle in a database like MySQL with InnoDB. The solution involves a combination of lock order management and cycle detection during commit or explicit locking operations. The key is to ensure that the transaction causing minimal disruption is rolled back, thereby resolving the deadlock without affecting more significant parts of the system.

#### Common Mistakes
and Follow-Ups
Missing Lock Order Management: Not enforcing consistent lock ordering can lead to unnecessary deadlocks.
Inadequate Cycle Detection: Failing to detect cycles early can result in prolonged blocking before resolution.
Incorrect Rollback Strategy: Choosing a transaction for rollback that doesn’t minimize impact could cause significant data inconsistencies.

