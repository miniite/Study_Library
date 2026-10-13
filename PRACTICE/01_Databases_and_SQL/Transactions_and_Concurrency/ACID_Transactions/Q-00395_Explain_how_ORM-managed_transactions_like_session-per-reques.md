---
type: "practice"
difficulty: "Deep"
---

# Explain how ORM-managed transactions (like session-per-request) can hide subtle isolation and flush-ordering bugs.

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Lock Holding and Optimistic Concurrency](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/ACID_Transactions/Lock_Holding_and_Optimistic_Concurrency.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a database development role, understanding ACID properties and transaction management is crucial. ORM-managed transactions, such as those often used in session-per-request patterns, can seem straightforward but hide subtle complexities related to isolation levels and flush orderings. This answer will break down these issues from first principles, highlighting the key concepts and potential pitfalls.

ORMs (Object-Relational Mappers) abstract away much of the database interaction, making them very convenient for developers. However, this abstraction comes with trade-offs. One such issue is how ORM-managed transactions can obscure subtle bugs related to transaction isolation and flush ordering. Let's explore these issues step-by-step from first principles.

**Comparison:**

```
User ID	Balance
1	0
```

**Code:**

```python
# Example of a simple session-per-request pattern using SQLAlchemy (a popular Python ORM)
from sqlalchemy import create_engine, text
from sqlalchemy.orm import Session

def process_request():
    engine = create_engine('sqlite:///example.db')
    with Session(engine) as session:
        # Simulate some database operations within the transaction
        session.execute(text("UPDATE users SET balance = 100 WHERE id = 1"))
        session.commit()  # Committing the transaction implicitly closes it
```

**Example input / output:**

```text
# Example input and output scenarios
input_request_1 = {"action": "update_balance", "user_id": 1}
input_request_2 = {"action": "update_balance", "user_id": 1}

output_response_1 = {
    "result": "Transaction started",
    # Transaction begins but is not yet committed
}

output_response_2 = {
    "result": "Transaction completed (commit)",
    # After this, the transaction in response_1 might be flushed or rolled back unexpectedly
}
```

**Data:**

```yaml
# Example of a simplified database schema for illustration purposes
database:
  tables:
    - name: users
      columns:
        - name: id (INTEGER, PRIMARY KEY)
        - name: balance (INTEGER)
```

**Flow:**

1. Yes
2. Yes
3. Yes
4. No
5. No
6. Request 1: Update balance
7. Is transaction started?
8. Transaction started internally
9. Database changes are notflushed yet
10. Is request 2 received?
11. Request 2: Update balance
12. Is transaction open for Request2?
13. Transaction continues, noissues
14. New transaction started forRequest 2
15. Database changes are notflushed yet
16. Transaction alreadycommitted/rolled back

