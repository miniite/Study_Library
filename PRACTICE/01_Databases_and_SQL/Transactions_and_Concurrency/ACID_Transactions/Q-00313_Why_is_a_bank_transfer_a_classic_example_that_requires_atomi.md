---
type: "practice"
difficulty: "Basic"
---

# Why is a bank transfer a classic example that requires atomicity?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Atomicity and Money Transfers](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/ACID_Transactions/Atomicity_and_Money_Transfers.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

A bank transfer is often used as an example of requiring atomicity because it involves multiple database operations (debits and credits) that must all succeed or fail together to maintain consistency in the banking system.

#### Triaging the Question

The question is probing your understanding of why certain operations, like a bank transfer, necessitate the ACID property known as atomicity. Atomicity ensures that a transaction is treated as an indivisible unit—either all changes are applied, or none at all. In the context of banking, this means ensuring that both debits and credits in a transfer occur simultaneously without external interference.

#### First-Principles Solution

Let's break down the steps to understand why atomicity is crucial for bank transfers:

Identify the Problem: A bank transfer involves moving funds from one account to another. This operation typically requires modifying two different accounts: the sender’s and the receiver’s.
Understand Atomicity: Atomicity means that each transaction must be either fully completed or entirely aborted, with no intermediate state. In other words, if a problem arises during the transfer (e.g., network failure), both parties should not see an incomplete change in their balances.

#### Example Scenario
Suppose Alice wants to transfer $100 from her account to Bob’s account.
The database must perform two operations: debit Alice's account and credit Bob's account.
Without Atomicity: If the system does not ensure atomicity, it could lead to inconsistent states. For example, if only one of these operations succeeds (e.g., Alice’s account is debited but Bob’s isn't), then both parties are left in an inconsistent state:
Alice now has less money, but there's no corresponding balance increase for Bob.
Atomicity Ensures Consistency: By ensuring that either both operations succeed or neither do, atomicity guarantees that the system remains in a consistent state.

#### Closing Summary
and Common Mistakes

In summary, bank transfers exemplify atomicity because they involve multiple database operations that must be treated as an indivisible unit to maintain consistency. Failure to ensure atomicity can result in inconsistent states, which could lead to financial discrepancies or even fraud.

#### Common Mistakes
A common mistake is assuming that individual operations are sufficient without considering the need for transactional integrity. Another mistake is underestimating the importance of ensuring all parts of a complex operation succeed together.

Depth + Language

For a fresher-level response, focus on the basics and provide concrete examples. For a mid-level response, you could delve into more technical details like database transactions and their implementation. At an architect level, you might discuss scalability concerns and distributed systems where ensuring atomicity can be even more complex.

**Comparison:**

```
Account ID	Username	Balance
1	Alice	400
2	Bob	500
Operation	Result
Start Transaction	Successful
Debit Alice's Account (-$100)	$400
Credit Bob's Account (+$100)	$500
Commit Transaction	Transaction Succeeded
```

**Code:**

```sql
-- Example SQL for a bank transfer
START TRANSACTION;

-- Debit Alice's account
UPDATE accounts SET balance = balance - 100 WHERE account_id = (SELECT id FROM users WHERE username = 'Alice');

-- Credit Bob's account
UPDATE accounts SET balance = balance + 100 WHERE account_id = (SELECT id FROM users WHERE username = 'Bob');

COMMIT;
```

**Example input / output:**

```text
Input:
- Transfer $100 from Alice to Bob

Output:
- Alice's balance after transfer: $400
- Bob's balance after transfer: $500
```

**Data:**

```json
{
  "accounts": [
    {
      "id": 1,
      "username": "Alice",
      "balance": 500
    },
    {
      "id": 2,
      "username": "Bob",
      "balance": 400
    }
  ]
}
```

**Flow:**

1. Start Transaction
2. Debit Alice's Account
3. Credit Bob's Account
4. Commit Transaction

