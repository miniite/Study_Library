---
type: "practice"
difficulty: "Deep"
---

# A transaction updates two rows in different tables and occasionally deadlocks with a reverse-order transaction — how do you enforce a canonical lock ordering in code?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Deadlocks in MySQL](<../../../../TOOLS/MySQL/MySQL_Platform/Deadlocks_in_MySQL.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview

When dealing with transactions that update multiple rows across different tables, ensuring consistent locking order can prevent deadlocks. In this scenario, we need to enforce a canonical lock ordering using MySQL to mitigate such issues. We'll explore the problem by understanding deadlock behavior, enforcing proper locking mechanisms, and discuss potential trade-offs.

#### Triaging the Question

The question is essentially about preventing database transactions from getting stuck in a deadlock state when updating multiple rows across different tables. The core issue here is ensuring that all transactions follow the same lock acquisition order to avoid cyclic dependency scenarios.

#### Assumptions
We are working with InnoDB engine, which supports row-level locking and allows explicit transaction control.
Transactions update two specific rows in different tables.

#### First Principles Solution

Understanding Deadlocks: A deadlock occurs when two or more transactions are waiting for each other to release resources they hold. For instance, if Transaction 1 locks Row X in Table A and waits for Row Y in Table B; while Transaction 2 locks Row Y in Table B and waits for Row X in Table A, both will hang indefinitely.

Defining Canonical Lock Order: To prevent deadlocks, we need to establish a canonical order of locking that all transactions must follow. For example, if the canonical order is "lock row X before row Y," then any transaction attempting to lock these rows must acquire them in this exact sequence.

#### Implementation
Steps:

Step 1: Identify the rows involved in each transaction.

SQL
-- Example tables and rows
CREATE TABLE table_a (id INT PRIMARY KEY, data VARCHAR(255));
CREATE TABLE table_b (id INT PRIMARY KEY, data VARCHAR(255));

INSERT INTO table_a VALUES (101, 'Data A');
INSERT INTO table_b VALUES (201, 'Data B');

Step 2: Write a function to enforce the canonical lock order in your application code. Here’s an example using Python with SQLAlchemy:

PYTHON
from sqlalchemy.orm import sessionmaker
from sqlalchemy import create_engine, Table, MetaData

engine = create_engine('mysql+pymysql://user:password@localhost/db_name')
Session = sessionmaker(bind=engine)
metadata = MetaData()
table_a = Table('table_a', metadata, autoload_with=engine)
table_b = Table('table_b', metadata, autoload_with=engine)

def enforce_lock_order(session, row_id_table_a, row_id_table_b):
    # Lock rows in the canonical order
    with session.begin():
        session.execute(table_a.update().where(table_a.c.id == row_id_table_a).values(column_name='new_value'))
        session.execute(table_b.update().where(table_b.c.id == row_id_table_b).values(column_name='new_value'))

session = Session()
enforce_lock_order(session, 101, 201)

Step 3: Ensure that all transactions follow the enforced locking order. This can be done by modifying your application logic to always lock resources in the same sequence.

Handling Deadlocks: In cases where a deadlock is detected (e.g., via LockTimeout or DeadlockDetected exceptions), you may need to retry the transaction. Alternatively, you could implement a timeout mechanism and roll back transactions that exceed the time limit.

#### Closing Summary

By enforcing a canonical lock ordering in your application code, you can significantly reduce the risk of deadlocks when dealing with multi-row updates across different tables. This approach ensures that all transactions follow a consistent sequence of locking resources, preventing cyclic dependencies and ensuring transactional consistency.

#### Common Mistakes/Follow-Ups
Failing to Enforce Locking Order: Not all developers or systems may be aware of the need for canonical lock ordering, leading to inconsistent behavior.
Timeouts and Retry Logic: Implementing retry logic with appropriate timeouts can help manage deadlocks but should be balanced against performance impacts.
Optimizing Performance: While enforcing a locking order helps prevent deadlocks, it might introduce additional overhead. You should monitor the application’s performance and adjust as necessary.

**Code:**

```python
from sqlalchemy.orm import sessionmaker
from sqlalchemy import create_engine, Table, MetaData

engine = create_engine('mysql+pymysql://user:password@localhost/db_name')
Session = sessionmaker(bind=engine)
metadata = MetaData()
table_a = Table('table_a', metadata, autoload_with=engine)
table_b = Table('table_b', metadata, autoload_with=engine)

def enforce_lock_order(session, row_id_table_a, row_id_table_b):
    with session.begin():
        session.execute(table_a.update().where(table_a.c.id == row_id_table_a).values(column_name='new_value'))
        session.execute(table_b.update().where(table_b.c.id == row_id_table_b).values(column_name='new_value'))

session = Session()
enforce_lock_order(session, 101, 201)
```

**Example input / output:**

| Input  | Output |
|--------|--------|
| `enforce_lock_order(Session(), 101, 201)` | Transaction locks row 101 in Table A and then row 201 in Table B. |

**Data:**

```sql
CREATE TABLE table_a (id INT PRIMARY KEY, data VARCHAR(255));
CREATE TABLE table_b (id INT PRIMARY KEY, data VARCHAR(255));

INSERT INTO table_a VALUES (101, 'Data A');
INSERT INTO table_b VALUES (201, 'Data B');
```

