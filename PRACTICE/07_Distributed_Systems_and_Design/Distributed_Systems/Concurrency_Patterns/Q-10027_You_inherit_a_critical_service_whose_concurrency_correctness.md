---
type: "practice"
difficulty: "Deep"
---

# You inherit a critical service whose concurrency correctness depends on undocumented tribal knowledge held by one departing senior engineer. As principal, how do you extract, formalize, and de-risk that knowledge (invariant docs, tests, guardrails) before they leave, and prevent this bus-factor pattern org-wide?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Distributed Coordination Choices](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Distributed_Coordination_Choices.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

As a backend architect tasked with ensuring the correctness of a critical service's concurrency patterns, I would prioritize extracting, formalizing, and de-risking any undocumented knowledge held by a departing senior engineer. The goal is to ensure that the service remains robust and maintainable, minimizing single points of failure (bus factors) and preventing similar issues in the future.

The key steps involve:

Gathering and Documenting Knowledge: We need to understand and document the existing concurrency patterns.
Formalizing Invariants: Define and codify these patterns into clear, testable invariants.
Implementing Guardrails: Add runtime checks and safety mechanisms to catch violations early.

First, let's break down the problem at hand:

#### Triage
What We're Probing

The question is probing our ability to:

Identify critical concurrency issues.
Document existing knowledge effectively.
Formalize and codify these patterns into testable invariants.
Implement safeguards against future failures.

#### Assumptions
The service involves complex concurrent operations, possibly with shared resources or state.
There are existing undocumented practices that ensure the service's correctness.
The senior engineer is leaving, and their knowledge is critical to maintaining the service’s integrity.
Step-by-Step Solution
1. Assessing Concurrency Patterns

The first step is to thoroughly assess the current concurrency patterns in the system. This involves understanding how shared resources are managed, data consistency rules, and any existing race conditions or synchronization mechanisms.

Worked Example: Consider a service that handles user transactions. Key operations include:

Depositing funds
Withdrawing funds
Querying account balances

These operations must ensure atomicity and consistency across multiple concurrent users.

2. Extracting Knowledge

To extract the knowledge, we need to interview the senior engineer and conduct a deep dive into their understanding of concurrency issues.

Code Extraction: We can start by looking at existing code snippets that handle these operations:

PYTHON
def deposit(amount):
    # Assume `account` is a shared resource managed via some thread-safe mechanism
    account.deposit(amount)

def withdraw(amount):
    if amount > 0 and account.balance >= amount:
        account.withdraw(amount)

Interview Approach: Conduct a structured interview to extract the engineer's understanding of how these operations are supposed to behave under various concurrency scenarios.

3. Documenting Invariants

Once we have a good understanding, document the invariants that must hold for correct operation:

Atomicity: Operations on shared resources should be treated as indivisible units.

#### Cons
istency: Shared state updates should maintain integrity across all concurrent accesses.
Race Condition Prevention: Ensure no race conditions can lead to incorrect states.

#### Example
Invariant Document:

MARKDOWN
## Concurrency Invariants for User Transactions

### Atomicity
- Deposits and Withdraws must be atomic. Any partial changes (e.g., depositing 50 out of 100) should not leave the system in an inconsistent state.

### Consistency
- Account balances must always reflect real-world financial transactions accurately, without any discrepancies due to race conditions.
4. Formalizing with Tests

Using formal methods and automated testing can help ensure that these invariants are met:

Unit Test Example:

PYTHON
import unittest

class TestTransactions(unittest.TestCase):
    def test_deposit(self):
        account = Account()
        initial_balance = 100
        expected_balance = initial_balance + 50
        account.deposit(50)
        self.assertEqual(account.balance, expected_balance)

    def test_withdraw(self):
        account = Account()
        initial_balance = 200
        amount_to_withdraw = 100
        if initial_balance >= amount_to_withdraw:
            account.withdraw(amount_to_withdraw)
        expected_balance = initial_balance - amount_to_withdraw
        self.assertEqual(account.balance, expected_balance)

if __name__ == '__main__':
    unittest.main()
5. Implementing Guardrails

Finally, we need to add runtime checks and safety mechanisms to catch violations early:

#### Example
Guardrail Code:

PYTHON
def deposit(amount):
    with account_lock:
        if account.balance + amount > max_balance:
            raise ValueError("Deposit would exceed maximum balance")
        account.deposit(amount)

def withdraw(amount):
    with account_lock:
        if amount > 0 and account.balance >= amount:
            account.withdraw(amount)

#### Closing Summary

By systematically extracting, formalizing, and implementing safeguards for the concurrency patterns, we can de-risk the critical service. This ensures that even after the senior engineer leaves, the system remains robust and maintainable.

#### Common Mistakes

Overlooking edge cases or race conditions.
Failing to document invariants clearly.
Not integrating test coverage effectively.
Final Self-check

The solution covers all key areas and addresses both the immediate need and long-term maintenance of the service.

**Comparison:**

```
Operation	Invariant
Deposit	Atomicity: Complete transaction.
Withdraw	Consistency: Balance reflects real-world transactions.
```

**Code:**

```python
def deposit(amount):
    with account_lock:
        if account.balance + amount > max_balance:
            raise ValueError("Deposit would exceed maximum balance")
        account.deposit(amount)

def withdraw(amount):
    with account_lock:
        if amount > 0 and account.balance >= amount:
            account.withdraw(amount)
```

**Example input / output:**

```text
# Test deposit operation
initial_balance = 100
amount_to_deposit = 50
expected_balance_after_deposit = initial_balance + amount_to_deposit

result = deposit(amount_to_deposit)
assert result == expected_balance_after_deposit, f"Expected balance {expected_balance_after_deposit}, got {result}"

# Test withdraw operation
initial_balance = 200
amount_to_withdraw = 100
expected_balance_after_withdraw = initial_balance - amount_to_withdraw

result = withdraw(amount_to_withdraw)
assert result == expected_balance_after_withdraw, f"Expected balance {expected_balance_after_withdraw}, got {result}"
```

**Data:**

```json
{
  "Account": {
    "balance": 100,
    "max_balance": 500
  },
  "TransactionTypes": ["deposit", "withdraw"]
}
```

**Flow:**

1. Yes
2. No
3. Transaction Initiated
4. Check Account Lock
5. Lock Acquired?
6. Perform Transaction
7. Unlock Account
8. Retry or Fail Transaction

