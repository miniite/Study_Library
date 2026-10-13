---
type: "practice"
difficulty: "Intermediate"
---

# How would you explain to a backend engineer why 'our ORM defaults to READ COMMITTED' matters for their money-transfer code?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Phantoms and Double Booking](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Phantoms_and_Double_Booking.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

When working with database transactions, understanding isolation levels is crucial for maintaining data integrity and performance, especially in a high-stakes application like money transfers. The ORM defaulting to READ COMMITTED can have significant implications on the behavior of your transactions and may lead to issues if not managed properly.

#### Triaging the Question

The question asks you to explain why an ORM defaulting to READ COMMITTED matters for backend engineers writing money-transfer code. Here’s what the interviewer is probing:

Isolation Levels Understanding: The candidate should be able to explain how different isolation levels (such as READ COMMITTED, REPEATABLE READ, SERIALIZABLE) work and their implications.
Transaction Isolation in Practice: How these settings affect transaction behavior, particularly in a scenario where data is being modified frequently, like in money transfers.
ORM Considerations: Why the ORM's default setting might need to be reconsidered for specific use cases.

#### First-Principles Solution
1. Understanding Isolation Levels

Isolation levels determine how transactions interact with each other and ensure that concurrent transactions do not see "dirty" data. Here are the four main isolation levels:

READ UNCOMMITTED: Allows reading uncommitted (dirty) data.
READ COMMITTED: Reads only committed data, but allows dirty reads if a transaction is modified before it commits.
REPEATABLE READ: Ensures that once a value has been read, subsequent reads of the same value will always return the same value, even if other transactions modify and commit values in between.
SERIALIZABLE: Provides the highest level of isolation by preventing dirty, non-repeatable, and phantom reads. It emulates serial execution order.
2. ORM Default Setting: READ COMMITTED

When an ORM defaults to READ COMMITTED, it means that any read operation will only see data that has been committed by other transactions. However, this setting can introduce subtle issues in money-transfer applications:

Lost Updates: If two transactions try to update the same row and one of them is READ COMMITTED, they might both attempt to write the same value, leading to a lost update scenario.
Dirty Reads: While rare with READ COMMITTED, it still allows for reading uncommitted data if multiple transactions modify the same rows concurrently.
3. Implications for Money Transfers

In a money-transfer application:

Concurrency Issues: If two users are transferring funds simultaneously, one transaction might read an outdated balance and then commit its changes before another transaction sees the updated value.

#### Cons
istency Problems: This can lead to inconsistent states where a user's account may show incorrect balances during the transaction process.
4. Best Practices

To ensure data consistency and avoid issues:

Use REPEATABLE READ or SERIALIZABLE Isolation Levels: These levels provide stronger guarantees against dirty reads, lost updates, and phantom reads.
Explicit Transaction Management: Use explicit transactions to manage the lifecycle of operations, ensuring that all relevant actions are executed atomically.

#### Closing Summary

Understanding isolation levels is critical for backend engineers working with applications like money transfers. The ORM defaulting to READ COMMITTED can introduce subtle but significant issues if not managed properly. For high-stakes financial transactions, it’s recommended to use higher isolation levels or explicitly manage transactions to ensure data consistency and integrity.

#### Common Mistakes
Not Recognizing the Need for Higher Isolation Levels: Engineers might stick with default settings without considering their implications.
Ignoring Concurrency Issues: Failing to account for concurrent updates can lead to critical bugs like lost updates.
Overlooking Transaction Management: Poorly managed transactions can result in inconsistent states and data anomalies.

**Comparison:**

```
Isolation Level	Dirty Reads	Non-repeatable Reads	Phantom Reads
READ UNCOMMITTED	Yes	Yes	Yes
READ COMMITTED	No	Yes	No
REPEATABLE READ	No	No	Yes
SERIALIZABLE	No	No	No
```

**Code:**

```python
# Example of how READ COMMITTED behaves with an ORM

from sqlalchemy.orm import sessionmaker, Session
from sqlalchemy import create_engine, Table, MetaData

engine = create_engine('sqlite:///example.db')
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

def transfer_money(sender_id: int, recipient_id: int, amount: float):
    with SessionLocal() as db:
        sender_account = db.query(Account).filter_by(id=sender_id).first()
        recipient_account = db.query(Account).filter_by(id=recipient_id).first()

        # READ COMMITTED might lead to a lost update
        if sender_account.balance > amount:
            sender_account.balance -= amount
            recipient_account.balance += amount
            db.commit()
```

**Example input / output:**

```text
Input:
- Transfer $200 from account with ID 1 to account with ID 2

Output:
- Account 1 balance becomes 300, but the transfer is lost if another transaction modifies Account 2's balance between reading and committing.
```

**Data:**

```json
{
  "Account": {
    "id": 1,
    "balance": 500.0
  }
}
```

**Flow:**

1. Transaction Starts
2. Reads Account Balance
3. Commits Changes
4. Isolation Level: READCOMMITTED
5. Another Transaction ReadsUncommitted Data
6. Conflict Occurs, Lost UpdatePossible

