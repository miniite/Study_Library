---
type: "practice"
difficulty: "Expert"
---

# How do long-running transactions hurt a database, and what is the risk of holding a transaction open across a network call?

**Difficulty:** Expert · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Isolation Anomalies](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Isolation_Anomalies.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Long-running transactions can significantly impact a database's performance and availability due to their exclusive locks on data records, which prevent other transactions from accessing or modifying those same records. When a transaction is held open across a network call, it adds an additional layer of risk, as the transaction state might become inconsistent if there are issues with the network connection. In this response, we'll first understand the core impact of long-running transactions and then discuss the risks associated with holding them open during network calls.

#### Triage
The question is essentially asking about two related but distinct concepts:

Impact of Long-Running Transactions: How such transactions can degrade database performance and availability.
Risk of Holding a Transaction Open Across Network Calls: What happens if a transaction's state becomes inconsistent due to network issues during its execution.

#### Assumptions
We'll assume the context is an SQL-based relational database management system (RDBMS) where transactions are managed using standard ACID properties—Atomicity, Consistency, Isolation, and Durability.

#### First-Principles Solution
Impact of Long-Running Transactions
Exclusive Locks: A transaction that holds locks on data records prevents other transactions from accessing or modifying the same records until it commits or rolls back. This can lead to a situation where multiple transactions are waiting, forming a deadlock.
Resource Contention: As more long-running transactions compete for database resources, such as CPU and memory, the performance of the entire system diminishes. Over time, this can degrade into a performance bottleneck.
Inconsistent Data: If a transaction is very long, there's an increased likelihood that changes made during its execution might be partially applied or corrupted due to external factors.
Risk of Holding a Transaction Open Across Network Calls
Network Issues: If a network call fails mid-transaction, the state of the transaction on both ends becomes inconsistent. For example, one side might have committed or rolled back a change, while the other hasn't.
Deadlocks and Rollbacks: The potential for deadlocks increases if network failures cause transactions to stall in different states.
Data Inconsistency: If data is updated but not fully propagated across nodes due to network issues, it can lead to inconsistent views of the database.
Mitigation Strategies
Short-Transaction Practices: Encourage developers to keep transactions as short-lived as possible by minimizing operations within a single transaction.
Network Resilience: Implement retry mechanisms and timeouts to handle transient network failures without leaving transactions in an ambiguous state.
Distributed Transaction Management: Use protocols like two-phase commit (2PC) or distributed transaction coordinators to manage consistency across multiple database nodes.

#### Example Scenario

#### Cons
ider a scenario where you have a web application making an API call that starts a long-running transaction. If this call fails due to a network error, the transaction state on both sides becomes unpredictable:

The client might think the transaction was successful.
The server might never receive confirmation of the operation's completion.

This can lead to data inconsistencies or even system failures if multiple such transactions are involved.

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, long-running transactions and holding them across network calls can introduce significant risks to database performance and consistency. Best practices include keeping transactions short-lived, implementing robust network error handling, and using distributed transaction management where necessary.

#### Common mistakes
include:

Ignoring the impact of long-running transactions on overall system performance.
Failing to implement proper error-handling mechanisms for network failures during transactions.
Not leveraging distributed transaction management systems in multi-node environments.

#### Follow-ups
could involve discussing specific tools or libraries that can help manage transactions more effectively, such as Oracle's Global Transaction Processing (GTP) or using a service mesh with built-in transactional guarantees.

**Code:**

```sql
-- Example of an SQL transaction block
BEGIN TRANSACTION;

-- Perform database operations here

COMMIT;  -- Commit the transaction to make changes permanent
ROLLBACK; -- Rollback in case of error
```

**Flow:**

1. Yes
2. No
3. Start Transaction
4. Network Call
5. Transaction Execution
6. Network Error?
7. Rollback Transaction
8. Commit Transaction
9. End Transaction

