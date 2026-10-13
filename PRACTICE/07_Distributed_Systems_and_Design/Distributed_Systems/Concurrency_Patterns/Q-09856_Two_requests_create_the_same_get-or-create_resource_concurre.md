---
type: "practice"
difficulty: "Deep"
---

# Two requests create the same "get-or-create" resource concurrently and you end up with duplicate rows despite a unique constraint being intended. Walk through the race and the idempotent upsert design that eliminates it.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Exactly Once via Idempotency](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Exactly_Once_via_Idempotency.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of duplicate rows created by concurrent "get-or-create" requests, we need to understand the root cause of the race condition and then implement a robust solution using idempotent upsert patterns. This problem is common in distributed systems where multiple clients might attempt to create or retrieve an entity with unique constraints.

#### Triage

The core issue here is a race condition that arises when two requests try to create the same resource concurrently, leading to duplicate entries despite the presence of a unique constraint. The interviewer is probing your understanding of concurrency patterns and race conditions in distributed systems.

#### Assumptions
We are dealing with a database system that supports transactions.
There is a unique constraint on the key field (e.g., id).
Concurrency control mechanisms like locks or atomic operations are available.

#### First-Principles Solution

To eliminate this race condition, we need to ensure that each "get-or-create" request is idempotent. Idempotency means that multiple identical requests should have the same effect as a single execution. This can be achieved by using a combination of read and write operations with appropriate concurrency control.

Step-by-Step Solution
Read: Attempt to read the resource from the database.
Check Existence: If the resource does not exist, proceed to create it; otherwise, return the existing resource immediately.
Write: Create or update the resource in a transactionally consistent manner.
Code Example

Here’s how you might implement this idempotent upsert pattern in Python using SQLAlchemy with PostgreSQL as the database:

PYTHON
from sqlalchemy import create_engine, Table, Column, Integer, String, MetaData, select, insert, update, and_
from sqlalchemy.orm import sessionmaker

# Database setup (simplified)
engine = create_engine('postgresql://user:password@localhost/dbname')
Session = sessionmaker(bind=engine)
session = Session()
metadata = MetaData()

resource_table = Table(
    'resource', metadata,
    Column('id', String, primary_key=True),
    Column('value', Integer)
)

def get_or_create(session, model, **kwargs):
    instance = session.query(model).filter_by(**kwargs).first()
    if instance:
        return instance
    else:
        session.add(model(**kwargs))
        session.commit()
        return session.query(model).filter_by(**kwargs).one()

# Example usage
resource_data = {'id': 'unique_id', 'value': 42}

with engine.connect() as conn:
    result = conn.execute(select([resource_table]).where(resource_table.c.id == resource_data['id']))
    if not result.fetchone():
        ins_stmt = insert(resource_table).values(**resource_data)
        update_stmt = (
            update(resource_table)
            .where(resource_table.c.id == resource_data['id'])
            .values(value=42)  # Update value as needed
            .returning(resource_table)
        )
        with conn.begin():  # Ensure atomicity
            ins_res = conn.execute(ins_stmt)
            if not ins_res.inserted_primary_key:
                upd_res = conn.execute(update_stmt)
                return upd_res.fetchone()
    else:
        return result.fetchone()

# Simplified idempotent upsert function
def upsert_resource(session, resource_data):
    instance = session.query(Resource).filter_by(id=resource_data['id']).first()
    if not instance:
        new_instance = Resource(**resource_data)
        session.add(new_instance)
        session.commit()
    return session.query(Resource).filter_by(id=resource_data['id']).one()

# Example of using the function
new_resource = {'id': 'unique_id', 'value': 42}
upserted_resource = upsert_resource(session, new_resource)

#### Closing Summary
and Common Mistakes/Follow-Ups

#### Closing Summary

The idempotent upsert pattern ensures that concurrent requests to create or retrieve a resource do not result in duplicate entries. By using transactions and atomic operations, we can guarantee the correctness of our database state even under high concurrency.

#### Common Mistakes/Follow-Ups
Ignoring Concurrency: Forgetting to handle race conditions, leading to duplicates.
Incorrect Transaction Management: Not ensuring that read and write operations are transactionally consistent.
Not Using Atomic Operations: Relying on optimistic locking or other methods without proper guarantees.

Additional questions might explore the trade-offs between different concurrency control mechanisms (e.g., pessimistic vs. optimistic locking), the impact of network latency, and how to handle retries in distributed systems.

**Comparison:**

```
Yes
No
Read
Resource Exists?
Return Existing Resource
Create New Resource
End
Write New Resource
```

**Code:**

```python
from sqlalchemy import create_engine, Table, Column, Integer, String, MetaData, select, insert, update, and_
```

**Example input / output:**

| Input | Output |
|-------|--------|
| `upsert_resource(session, new_resource)` with `new_resource = {'id': 'unique_id', 'value': 42}` | Returns the upserted resource object. If the resource already exists, returns the existing instance; otherwise, creates a new one and returns it. |

**Data:**

```json
{
    "id": "unique_id",
    "value": 42
}
```

**Flow:**

1. Yes
2. No
3. Read
4. Resource Exists?
5. Return Existing Resource
6. Create New Resource
7. End
8. Write New Resource

