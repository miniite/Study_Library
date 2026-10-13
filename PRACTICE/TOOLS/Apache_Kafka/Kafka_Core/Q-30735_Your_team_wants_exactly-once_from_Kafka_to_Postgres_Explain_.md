---
type: "practice"
difficulty: "Deep"
---

# Your team wants exactly-once from Kafka to Postgres. Explain why the Kafka transaction alone is insufficient and how you'd make the DB write idempotent instead.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Exactly Once Beyond Kafka](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Exactly_Once_Beyond_Kafka/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure exactly-once processing (EOP) when writing data from Kafka to PostgreSQL, simply relying on Kafka's transactions is not sufficient because they only address the consistency within Kafka's internal state and do not guarantee idempotent writes to the database. We need to make sure that each message in Kafka can be processed by the application at most once, even if the application retries due to failures.

#### Triage
What the Question is Asking

The interviewer is probing your understanding of exactly-once processing (EOP) and how it extends beyond just ensuring consistent state changes within Kafka. Specifically, they want you to explain why Kafka’s transaction semantics are not enough for EOP when writing to a relational database like PostgreSQL.

Step-by-Step Solution from First Principles
Understand Kafka's Transaction Semantics: Kafka transactions ensure that messages in the same transaction batch are processed atomically. If a consumer fails during processing, it can retry and redeliver the entire batch, ensuring exactly-once delivery of messages within Kafka itself.
Identify Limitations with Kafka Transactions: However, Kafka transactions only control the state within Kafka; they do not prevent idempotent writes to external databases like PostgreSQL. A transaction in Kafka does not guarantee that a message is processed by the application at most once.
Make DB Writes Idempotent: To achieve exactly-once processing when writing from Kafka to PostgreSQL, we need to ensure that each database write is idempotent—meaning it can be safely retried without causing unintended side effects.
Making Database Writes Idempotent
Identify the Key Data Structure: Let's assume we are inserting or updating a record in a table with a primary key (PK) constraint.
Use Unique Constraints and Versioning:
Version Number Approach: Add a version number column to your table that gets incremented on each update. When processing a message, check the current version of the record before performing an update. If the version has changed since you last checked, skip the operation; otherwise, proceed with the update.
Unique Constraint Approach: Use unique constraints or composite keys in PostgreSQL to ensure that duplicate records are not inserted.

#### Example
Code
SQL
-- Define a table with a version number and unique constraint
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT NOT NULL,
    amount DECIMAL(10, 2) NOT NULL,
    version INT DEFAULT 1 NOT NULL,
    UNIQUE (order_id, version)
);
Handling Kafka Messages

#### Cons
umer Logic:
For each message from Kafka, extract the order_id and amount.
Use a database client to perform an upsert operation (INSERT or UPDATE) based on the presence of a row with that order_id and version.
Idempotent Processing:
PYTHON
def process_order_message(message):
    order_id = message['order_id']
    amount = message['amount']

    # Check if the record exists with the current version
    result = db.execute(
        "SELECT * FROM orders WHERE order_id = :order_id AND version = 1",
        {"order_id": order_id}
    )

    if not result.fetchone():
        # Insert a new row if it doesn't exist yet
        db.execute(
            "INSERT INTO orders (order_id, customer_id, amount) VALUES (:order_id, :customer_id, :amount)",
            {"order_id": order_id, "customer_id": 12345, "amount": amount}
        )
    else:
        # Update the existing row
        db.execute(
            "UPDATE orders SET amount = :new_amount WHERE order_id = :order_id AND version = 1",
            {"order_id": order_id, "new_amount": new_amount}
        )

# Assuming db is a database connection object

#### Closing Summary
and Common Mistakes/Follow-Ups

By making the database writes idempotent through mechanisms like unique constraints or versioning, we ensure that each message from Kafka is processed at most once by the application. This approach complements Kafka's transactional semantics and provides the necessary exactly-once processing guarantees.

#### Common Mistakes
to Avoid:

Failing to handle concurrent updates correctly.
Not properly managing transaction boundaries in a distributed system.
Overlooking the need for idempotent writes when dealing with retries.

If asked further, you might discuss more advanced topics like distributed transactions using Kafka's transactional semantics across multiple topics or even leveraging external coordination services like Zookeeper or Kafka’s own inter-broker protocol to manage distributed state.

