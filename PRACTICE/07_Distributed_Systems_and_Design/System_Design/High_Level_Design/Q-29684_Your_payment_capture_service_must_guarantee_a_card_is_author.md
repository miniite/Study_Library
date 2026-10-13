---
type: "practice"
difficulty: "Deep"
---

# Your payment capture service must guarantee a card is authorized then captured exactly once even if the auth response is lost on the network — walk through the state machine reconciliation query and timeout handling you'd build.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Payment Ledgers](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Payment_Ledgers.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that a card is authorized and captured exactly once, even if the authorization response is lost on the network, we need to design a robust state machine with careful consideration of reconciliation queries and timeout handling. The core idea is to maintain a consistent state across multiple systems while ensuring that each transaction is processed only once.

In this scenario, the payment capture service will handle two primary states: AUTHORIZE and CAPTURE. We'll use a database to store these states along with timestamps for reconciliation purposes. Additionally, we'll implement timeouts to prevent indefinite waits and ensure timely retries or failures.

#### Triage

The question is essentially asking how to design a reliable state machine that ensures exactly-once processing of transactions in the presence of network loss. This involves:

Defining the transaction states (AUTHORIZE and CAPTURE).
Managing the state transitions with atomic updates.
Implementing timeout mechanisms for retries or failures.
Handling reconciliation queries to ensure consistency across systems.

#### Assumptions
The database supports transactions that can be used for atomic operations.
Network loss is a possibility but not expected frequently.
We need to handle both primary and backup systems in case of failures.

#### First-Principles Solution

State Machine Design:

AUTHORIZE: When the card is first presented, we set the state to AUTHORIZE.
CAPTURE: Once the transaction is confirmed as authorized, we transition to the CAPTURE state.

Database Schema:

SQL
CREATE TABLE Transactions (
    id SERIAL PRIMARY KEY,
    card_id UUID NOT NULL,
    status VARCHAR(50) CHECK (status IN ('AUTHORIZE', 'CAPTURE')),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

State Transition Logic:

When a request comes in to authorize a card:
SQL
START TRANSACTION;
UPDATE Transactions SET status = 'AUTHORIZE', updated_at = NOW() WHERE card_id = <card_id> AND status = 'INITIAL';
IF ROW_COUNT() > 0 THEN
    COMMIT;
ELSE
    ROLLBACK;
END IF;
When a request comes in to capture the transaction:
SQL
START TRANSACTION;
UPDATE Transactions SET status = 'CAPTURE', updated_at = NOW() WHERE card_id = <card_id> AND status = 'AUTHORIZE';
IF ROW_COUNT() > 0 THEN
    COMMIT;
ELSE
    ROLLBACK;
END IF;

Timeout Handling:

Set a timeout value (e.g., 5 minutes) to automatically retry failed transactions.
Implement a background worker that periodically checks for pending transactions and retries them if the state hasn't changed.

Reconciliation Queries:

To handle lost messages, we can use reconciliation queries that compare states between primary and backup systems.
SQL
SELECT * FROM Transactions WHERE status IN ('AUTHORIZE', 'CAPTURE') AND updated_at < NOW() - INTERVAL 1 HOUR;
If a discrepancy is found, update the state to ensure consistency.

#### Closing Summary
+ Common Mistakes/Follow-Ups

#### Common Mistakes

Failing to use transactions for atomic operations.
Not handling timeouts and retries properly, leading to indefinite waits or data inconsistencies.
Ignoring state transitions logic, which can lead to race conditions.

#### Follow-Ups

How would you handle concurrent transactions?
What if a card is authorized multiple times?
How do you ensure the system scales with increasing transaction volumes?

**Comparison:**

| id       | card_id             | status   | created_at                 | updated_at                 |
|----------|---------------------|----------|----------------------------|----------------------------|
| 1        | 123e4567-e89b-12d3-a456-426614174000 | INITIAL  | 2023-10-01 12:00:00 UTC    | 2023-10-01 12:00:00 UTC    |
| 2        | 567e89b-12d3-a456-426614174000   | AUTHORIZE| 2023-10-02 12:00:00 UTC    | 2023-10-02 12:00:00 UTC    |

**Code:**

```sql
-- Database Schema
CREATE TABLE Transactions (
    id SERIAL PRIMARY KEY,
    card_id UUID NOT NULL,
    status VARCHAR(50) CHECK (status IN ('INITIAL', 'AUTHORIZE', 'CAPTURE')),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

-- Authorize Transaction
START TRANSACTION;
UPDATE Transactions SET status = 'AUTHORIZE', updated_at = NOW() WHERE card_id = <card_id> AND status = 'INITIAL';
IF ROW_COUNT() > 0 THEN
    COMMIT;
ELSE
    ROLLBACK;
END IF;

-- Capture Transaction
START TRANSACTION;
UPDATE Transactions SET status = 'CAPTURE', updated_at = NOW() WHERE card_id = <card_id> AND status = 'AUTHORIZE';
IF ROW_COUNT() > 0 THEN
    COMMIT;
ELSE
    ROLLBACK;
END IF;
```

**Example input / output:**

```text
-- Input: Authorize Transaction
START TRANSACTION;
UPDATE Transactions SET status = 'AUTHORIZE', updated_at = NOW() WHERE card_id = '123e4567-e89b-12d3-a456-426614174000' AND status = 'INITIAL';
-- Output: (Rows matched: 1)

-- Input: Capture Transaction
START TRANSACTION;
UPDATE Transactions SET status = 'CAPTURE', updated_at = NOW() WHERE card_id = '123e4567-e89b-12d3-a456-426614174000' AND status = 'AUTHORIZE';
-- Output: (Rows matched: 1)
```

**Data:**

```sql
-- Sample Data
INSERT INTO Transactions (card_id, status) VALUES ('123e4567-e89b-12d3-a456-426614174000', 'INITIAL');
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Yes
6. No
7. Receive Authorization Request
8. Card ID Exists?
9. Update Status to AUTHORIZE
10. Log Error and Fail
11. Status is INITIAL?
12. Commit Transaction
13. Receive Capture Request
14. Card ID Exists?
15. Update Status to CAPTURE
16. Log Error and Fail
17. Commit Transaction

