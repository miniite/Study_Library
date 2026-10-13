---
type: "practice"
difficulty: "Deep"
---

# Explain how an ORM's implicit transaction boundaries and lazy loading can create deadlocks that are invisible in the application code.

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Deadlock Causes and Diagnosis](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/Deadlocks/Deadlock_Causes_and_Diagnosis.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with Object-Relational Mapping (ORM) tools, developers often rely on their abstractions to handle database interactions efficiently. However, these abstractions can introduce subtle issues like deadlocks that are not immediately apparent in the application code. In this answer, we will explore how ORM's implicit transaction boundaries and lazy loading mechanisms can create invisible deadlocks.

First, let’s clarify what a deadlock is: it occurs when two or more processes wait for each other to release resources, resulting in a circular dependency. In an ORM context, this can happen without the application explicitly managing transactions.

To understand how these issues arise, we need to break down the interactions between the ORM and the database.

#### Triaging the Question

The question is probing our understanding of how implicit transaction management and lazy loading can lead to deadlocks that are not easily detectable in application code. This involves identifying where deadlocks might occur due to ORM’s automatic handling of transactions, which often includes multiple statements executed across different points in time or under different scopes.

#### Assumptions
ORM Mechanisms: We assume the ORM automatically starts and commits transactions at certain points during object persistence.
Lazy Loading: The application retrieves a part of an entity graph eagerly while others are loaded lazily, potentially causing delayed interactions with the database.
Concurrency: Multiple concurrent operations running in different threads or processes.

#### First-Principles Solution
Implicit Transaction Boundaries

ORMs often use implicit transaction boundaries to ensure that changes made through the ORM are atomic and consistent. For example, when an entity is modified, a session manages this change until it commits. However, if multiple entities are involved in different operations within the same application logic, these operations might be treated as separate transactions.

#### Cons
ider a scenario where Entity A depends on Entity B. The application modifies Entity A, which implicitly starts a transaction. Meanwhile, another operation occurs that needs to load Entity B lazily. If both of these actions happen concurrently and Entity B is part of the transaction started by modifying Entity A, a deadlock can occur if both transactions are waiting for each other.

Lazy Loading

Lazy loading ensures that database queries are deferred until necessary. This can complicate deadlock detection because the actual database interactions might not be evident in the application flow. For instance, consider an operation where we first load an entity partially and then later retrieve related entities through lazy loading. These subsequent operations could be executed under different transaction contexts than the initial one.

#### Example Scenario

Imagine a banking system with two accounts: Account A and Account B. The application tries to transfer money from A to B in two steps:

Load Account A (lazy-loaded).
Deduct funds from Account A.
Load Account B (lazy-loaded).
Add funds to Account B.

If these steps are executed concurrently, each operation might start a transaction implicitly. If Account B’s balance update waits for the commit of Account A’s deduction but Account A’s deduction is waiting for the commit of Account B's addition, both transactions will deadlock.

#### Closing Summary
and Common Mistakes

In summary, ORM's implicit transaction boundaries and lazy loading can create deadlocks that are invisible in application code because:

Different parts of the same operation might be treated as separate transactions.
Lazy-loaded data introduces delayed database interactions, making it hard to trace the actual operations causing the deadlock.

#### Common mistakes
include underestimating the complexity introduced by ORM's automatic transaction handling and not considering how lazy loading can lead to delayed interactions that are difficult to predict and debug.

Common Follow-Ups
How would you detect such deadlocks?
What strategies can be used to prevent or resolve these deadlocks in an ORM environment?
How does this issue differ from explicit transaction management using raw SQL or a database client library?

**Comparison:**

```
Operation	Thread 1	Thread 2
Step 1	Load Account A	Load Account B
Step 2	Deduct funds	Add funds
```

**Code:**

```python
# Example of lazy loading and implicit transactions
from sqlalchemy.orm import sessionmaker, Session

Session = sessionmaker()
session: Session

class Account:
    @property
    def balance(self):
        # Lazy-loaded property that triggers a DB query when accessed
        if not hasattr(self, '_balance'):
            self._balance = session.query(Balance).filter_by(account_id=self.id).first().amount
        return self._balance

# Transaction boundaries are implicit in ORM operations
account_a: Account
account_b: Account

session.begin()  # Implicit transaction starts here
try:
    account_a.balance -= 100  # Lazy loading triggers a DB query, implicitly starting another transaction
    session.commit()
except Exception as e:
    pass  # Handle exceptions

try:
    account_b.balance += 100  # Another implicit transaction started
    session.commit()
except Exception as e:
    pass  # Handle exceptions
```

**Example input / output:**

```text
Input 1 (Thread 1):
- Load Account A (Lazy)
- Deduct funds from Account A

Input 2 (Thread 2):
- Load Account B (Lazy)
- Add funds to Account B

Output:
Deadlock detected on transactions involving accounts and balances.
```

**Data:**

```yaml
accounts:
- id: 1
  name: Account A
- id: 2
  name: Account B

balance:
- account_id: 1
  amount: 500
- account_id: 2
  amount: 300
```

**Flow:**

1. No
2. No
3. Load Account A
4. Is Balance Loaded?
5. Deduct Funds from A
6. Commit Transaction
7. Load Account B
8. Is Balance Loaded?
9. Add Funds to B
10. Commit Transaction
11. Transaction Deadlock Detected

