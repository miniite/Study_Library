---
type: "practice"
difficulty: "Deep"
---

# Explain why a distributed lock is often the wrong abstraction for a database job and when you'd instead make the operation idempotent or use a fencing token. Give a concrete SQL example for each choice.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Distributed Locks and Fencing](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Idempotent_Ingestion/Distributed_Locks_and_Fencing.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A distributed system can be complex due to its inherent challenges such as consistency, availability, and partition tolerance (CAP theorem). Distributed locks are often seen as an elegant solution to ensure exclusive access to a resource across multiple nodes. However, they can lead to several issues, including deadlocks, lock contention, and performance bottlenecks. In contrast, making operations idempotent or using fencing tokens can provide robust solutions that avoid these pitfalls.

To illustrate the scenarios where distributed locks might not be suitable and how idempotency or fencing tokens could be used instead, let's consider a concrete example: updating user balances in a financial transaction system.

TL;DR

Distributed locks are often overly complex for database operations. Idempotent operations or fencing tokens can provide simpler and more reliable solutions by ensuring that the operation is safe to retry multiple times without causing unintended side effects.

#### Triaging the Question

The question asks you to explain why a distributed lock might be unsuitable in certain contexts, such as updating user balances in a financial transaction system. The interviewer is probing your understanding of the limitations and potential downsides of using distributed locks versus other approaches like idempotency or fencing tokens. Key assumptions include:

You have experience with distributed systems.
You understand the implications of consistency, availability, and partition tolerance.

#### First-Principles Solution
Why Distributed Locks Are Often Unnecessary

Distributed locks can introduce significant complexity into a system due to their overhead and potential for deadlock or livelock scenarios. In a financial transaction system, the balance update operation is often idempotent by nature, meaning it can safely be retried without changing its outcome.

#### Example
of Idempotent Operation: Consider an SQL statement that updates a user's balance:

SQL
UPDATE users SET balance = balance + 10 WHERE user_id = '123';

This operation is idempotent because adding the same amount to the balance multiple times will not change the final result. If this operation fails and retries, it will simply add another $10 to the balance.

Fencing Tokens

Fencing tokens are a more advanced technique used to ensure that only one node can perform an operation at a time by using a unique identifier (token) for each transaction. This avoids lock contention and reduces the risk of deadlocks.

#### Example
with Fencing Token: Using a fencing token in SQL could look like this:

SQL
BEGIN TRANSACTION;
SELECT balance INTO @temp_balance FROM users WHERE user_id = '123' FOR UPDATE;
-- Use @temp_balance to process the transaction logic.
UPDATE users SET balance = @temp_balance + 10 WHERE user_id = '123';
COMMIT;

In this case, the FOR UPDATE clause ensures that only one node can access and modify the balance at a time. The token here is implicitly created by the database itself or managed through application-level logic.

#### Closing Summary

Distributed locks are powerful but often unnecessary for simple operations like updating user balances in a financial transaction system. Idempotent operations ensure safety without requiring complex locking mechanisms, while fencing tokens provide robust solutions to ensure exclusive access when necessary.

#### Common Mistakes
and Follow-Ups
Mistake: Overusing distributed locks for every operation, leading to increased complexity.

#### Follow-Up
Discuss how to identify whether an operation is idempotent or if a fencing token might be needed. Consider edge cases like concurrent transactions and the impact on performance.

**Comparison:**

```
Operation	Idempotent Example	Fencing Token Example
SQL	UPDATE ...	BEGIN TRANSACTION; SELECT balance INTO ... FOR UPDATE; SET @fence_token = UUID(); UPDATE ...; COMMIT;
```

**Code:**

```sql
-- Idempotent Operation Example
UPDATE users SET balance = balance + 10 WHERE user_id = '123';

-- Fencing Token Example (using FOR UPDATE)
BEGIN TRANSACTION;
SELECT balance INTO @temp_balance FROM users WHERE user_id = '123' FOR UPDATE;
SET @fence_token = UUID(); -- Generate a unique token
UPDATE users SET balance = @temp_balance + 10, fence_token = @fence_token WHERE user_id = '123';
COMMIT;
```

**Example input / output:**

```

Input:

SQL
Copy
UPDATE users SET balance = balance + 10 WHERE user_id = '123';

Output:

SQL
Copy
BEGIN TRANSACTION;
SELECT balance INTO @temp_balance FROM users WHERE user_id = '123' FOR UPDATE;
SET @fence_token = UUID();
UPDATE users SET balance = @temp_balance + 10, fence_token = @fence_token WHERE user_id = '123';
COMMIT;
```

**Data:**

```json
{
  "user_id": "123",
  "balance": 500,
  "transaction_amount": 10,
  "fence_token": "abc-123-def"
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Idempotent Operation
6. Is the operation idempotent?
7. Use simple SQL update
8. Fencing Token Required?
9. Generate Fence Token
10. Update with Token Check
11. Use Distributed Lock
12. Transaction Commit
13. Transaction Commit

