---
type: "practice"
difficulty: "Expert"
---

# Design a payment ledger / double-entry accounting system — how do you guarantee balances are always consistent and every transaction is auditable and replayable?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Payment Ledgers](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Payment_Ledgers.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

To design a high-level payment ledger that guarantees consistency in account balances, ensures auditability, and supports replayability of transactions, we need to implement a robust double-entry accounting system with key components like transaction logging, atomic operations, distributed consensus, and immutable logs. This solution leverages database integrity constraints, transactional guarantees, and blockchain-inspired techniques for traceability.

#### Triaging the Question

The interviewer is probing your ability to design a highly reliable financial system that can handle transactions efficiently while ensuring data consistency, auditability, and replayability. The core of this question involves understanding how to manage state changes in a distributed environment and implement mechanisms to detect and correct inconsistencies.

#### Assumptions
Transactions: Each transaction involves debiting one account and crediting another.

#### Cons
istency: All transactions must be applied atomically to maintain balance consistency.
Auditability: Every transaction should be traceable, timestamped, and loggable.
Replayability: Transactions can be replayed in any order without affecting the outcome.

#### Solution
Step 1: Data Model

We will use a relational database or a distributed ledger to store transactions. Each transaction is represented as an entity with fields like id, sourceAccountId, destinationAccountId, amount, and timestamp.

SQL
CREATE TABLE Transactions (
    id BIGINT PRIMARY KEY,
    sourceAccountId INT NOT NULL,
    destinationAccountId INT NOT NULL,
    amount DECIMAL(10, 2) NOT NULL,
    timestamp TIMESTAMP NOT NULL
);
Step 2: Atomic Operations with Databases

To ensure atomicity, use database transactions. Begin a transaction before starting the operation and commit/roll back based on success/failure.

SQL
BEGIN TRANSACTION;
UPDATE Accounts SET balance = balance - amount WHERE id = sourceAccountId;
UPDATE Accounts SET balance = balance + amount WHERE id = destinationAccountId;
COMMIT;
Step 3: Consistency Checks with Invariants

Implement database invariants to ensure that no account can have a negative balance.

SQL
CREATE CONSTRAINT CHECK (balance >= 0);
Step 4: Audit Trail

Log all transactions along with metadata for future reference. This could be implemented using a separate audit table or even a blockchain for added security and immutability.

SQL
INSERT INTO AuditLogs (transactionId, accountId, action, timestamp) VALUES (id, sourceAccountId, 'debit', timestamp);
INSERT INTO AuditLogs (transactionId, accountId, action, timestamp) VALUES (id, destinationAccountId, 'credit', timestamp);
Step 5: Replayability

To replay transactions, simply read the logs and apply them in order. This ensures that even if a system fails, it can be recovered by re-executing all transactions.

#### Closing Summary

In summary, to design a high-level payment ledger, we need to ensure atomic operations using database transactions, enforce balance invariants, maintain an audit trail for traceability, and support replayability through log-based recovery. This approach guarantees that the system remains consistent and auditable, even under failure scenarios.

#### Common Mistakes/Follow-Ups
Failing to Use Transactions: Not leveraging ACID properties can lead to inconsistent states.
Ignoring Invariants: Neglecting balance checks can result in negative balances which violate financial integrity.
Lack of Traceability: Not maintaining logs can make debugging and auditing extremely difficult.

Would you like me to elaborate on any specific aspect, such as implementing distributed consensus or optimizing performance?

**Comparison:**

```
Accounts
Transactions
AuditLogs
```

**Code:**

```sql
-- SQL for creating tables
CREATE TABLE Accounts (
    id INT PRIMARY KEY,
    balance DECIMAL(10, 2)
);

CREATE TABLE Transactions (
    id BIGINT PRIMARY KEY,
    sourceAccountId INT NOT NULL,
    destinationAccountId INT NOT NULL,
    amount DECIMAL(10, 2) NOT NULL,
    timestamp TIMESTAMP NOT NULL
);

-- SQL for creating constraints and invariants
CREATE CONSTRAINT CHECK (balance >= 0);
```

**Example input / output:**

| Input Transactions |
|-------------------|
| { "sourceAccountId": 1, "destinationAccountId": 2, "amount": 30.00 } |
| { "sourceAccountId": 2, "destinationAccountId": 1, "amount": 20.00 } |
| Output Accounts |
|----------------|
| Account 1: 90.00 |
| Account 2: 70.00 |
| Audit Logs (after transactions) |
|---------------------------------|
| { "transactionId": 1, "accountId": 1, "action": "debit", "timestamp": "2023-10-01T14:00" } |
| { "transactionId": 1, "accountId": 2, "action": "credit", "timestamp": "2023-10-01T14:00" } |
| { "transactionId": 2, "accountId": 2, "action": "debit", "timestamp": "2023-10-02T15:00" } |
| { "transactionId": 2, "accountId": 1, "action": "credit", "timestamp": "2023-10-02T15:00" } |

**Data:**

```json
{
  "Accounts": [
    { "id": 1, "balance": 100.00 },
    { "id": 2, "balance": 50.00 }
  ],
  "Transactions": [
    { "id": 1, "sourceAccountId": 1, "destinationAccountId": 2, "amount": 30.00, "timestamp": "2023-10-01T14:00" },
    { "id": 2, "sourceAccountId": 2, "destinationAccountId": 1, "amount": 20.00, "timestamp": "2023-10-02T15:00" }
  ]
}
```

**Flow:**

1. Log
2. AC
3. DB
4. User
5. Log
6. AC
7. DB
8. User
9. Initiate transaction
10. Acquire lock on accounts
11. Perform debit and credit operations
12. Commit transactions
13. Update balances
14. Write audit logs
15. Confirm success

