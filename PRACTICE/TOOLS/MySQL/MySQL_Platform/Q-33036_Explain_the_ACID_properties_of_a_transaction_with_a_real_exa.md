---
type: "practice"
difficulty: "Intermediate"
---

# Explain the ACID properties of a transaction with a real example.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Reading EXPLAIN](<../../../../TOOLS/MySQL/MySQL_Platform/Reading_EXPLAIN.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with databases, understanding the ACID (Atomicity, Consistency, Isolation, Durability) properties is crucial to ensure reliable and consistent data transactions. Let's dive into each of these properties through a real-world example using MySQL.

#### Overview

ACID properties are a set of standards that define the conditions required for a database transaction to be executed correctly without any external interference or unexpected issues. In this explanation, we will use an example where two users simultaneously try to withdraw and deposit money from/to each other's bank accounts in a bank management system.

#### Triaging the Question

The question is essentially asking you to explain ACID properties with a practical example. The interviewer is probing your understanding of transactional integrity, how these properties interact, and their importance in ensuring database reliability. You need to assume that we are working within an environment where MySQL is used as the database management system.

#### First-Principles Solution
Atomicity: Ensures that all operations within a transaction are treated as a single unit of work; if one part fails, none should succeed.

#### Cons
istency: Guarantees that any transaction must move the database from one valid state to another without violating constraints.
Isolation: Prevents interference between concurrent transactions by ensuring that each transaction sees only the data it should see.
Durability: Ensures that once a transaction has been committed, its changes will remain, even in case of system failures.

#### Example Scenario

Suppose we have two users, Alice and Bob, who are trying to exchange money using our bank management application. Both are performing transactions simultaneously: Alice is withdrawing 100 dollars from her account and depositing the same amount into Bob's account.

SQL
-- User Accounts Table Schema
CREATE TABLE accounts (
    id INT PRIMARY KEY,
    balance DECIMAL(10,2) NOT NULL
);
Step-by-Step Walkthrough:

Transaction Start:

Both transactions start at almost the same time.

Alice Withdraws 100$ from her Account:

SQL
BEGIN;
SELECT balance FROM accounts WHERE id = 1 FOR UPDATE; -- lock Alice's account to prevent concurrent modifications
SET balance = balance - 100.00;

Bob Deposits 100$ into his Account (assuming Bob has a different ID, say 2):

SQL
BEGIN;
SELECT balance FROM accounts WHERE id = 2 FOR UPDATE; -- lock Bob's account to prevent concurrent modifications
SET balance = balance + 100.00;

Transaction Conflict:

Both transactions now try to modify the same state (balance of one account), but within their own transactions.

Atomicity Ensures Integrity:

If both updates are successful, the transaction is committed, and all operations within it are executed as a single unit of work. If either fails, none should succeed.

#### Cons
istency Maintained:

The transactions ensure that the balance changes adhere to the constraints (e.g., no negative balances).

Isolation Ensures No Interference:

The use of FOR UPDATE locks prevent other transactions from modifying these accounts during the transaction, ensuring isolation.

Durability Ensures Persistence:

Once a transaction is committed, changes are permanently saved to the database.

#### Closing Summary
and Common Mistakes/Follow-Ups:

In our example, if both transactions were atomic, consistent, isolated, and durable, they would ensure that the final state of the accounts (after processing) remains valid. Any failure in any part of this process could lead to inconsistent states or data corruption, which is why understanding ACID properties is so crucial.

#### Common mistakes
include not properly handling concurrent transactions, ignoring constraints during updates, failing to commit/rollback appropriately, and not recognizing the importance of isolation levels in preventing race conditions.

#### Follow-ups
might explore different database configurations, transactional behavior with different isolation levels, or specific failure scenarios (e.g., what happens if a power outage occurs just after the COMMIT).

**Code:**

```sql
BEGIN;
SELECT balance FROM accounts WHERE id = 1 FOR UPDATE; -- lock Alice's account to prevent concurrent modifications
SET balance = balance - 100.00;

BEGIN;
SELECT balance FROM accounts WHERE id = 2 FOR UPDATE; -- lock Bob's account to prevent concurrent modifications
SET balance = balance + 100.00;
```

**Data:**

```sql
CREATE TABLE accounts (
    id INT PRIMARY KEY,
    balance DECIMAL(10,2) NOT NULL
);
```

