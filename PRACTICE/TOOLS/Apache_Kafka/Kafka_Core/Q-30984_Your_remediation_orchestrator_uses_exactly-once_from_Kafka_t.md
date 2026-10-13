---
type: "practice"
difficulty: "Expert"
---

# Your remediation orchestrator uses exactly-once from Kafka to Postgres for the action-audit log. Explain why the Kafka transaction alone isn't enough and how you make the DB write idempotent.

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ Exactly Once Beyond Kafka](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Exactly_Once_Beyond_Kafka/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that the remediation orchestrator can reliably process actions in exactly-once semantics, using Kafka is a good start because it provides guaranteed delivery and at-least-once processing. However, to achieve exactly-once semantics, especially when writing to Postgres from Kafka, we need to make sure both the Kafka transaction and the database write are idempotent or compensatable. The Kafka transaction alone isn't sufficient for ensuring idempotency in the database writes. Let's break down why and how we can address this.

#### Triaging the Question

The question is probing our understanding of exactly-once semantics, the limitations of Kafka transactions, and how to ensure idempotent writes to Postgres. We need to assume that the orchestrator handles failures by retrying actions and possibly needing to compensate for them if they fail or are incorrectly processed.

#### First Principles Solution

Kafka Transactions: Kafka's transaction guarantees provide at-least-once delivery, meaning a message will be delivered at least once but may be delivered multiple times. This is essential because it ensures that messages are not lost in case of failures.

Idempotent Writes to Postgres: The issue with Kafka transactions alone is that they only guarantee the order and delivery of messages within a transaction, not the idempotency or compensatability of the database writes themselves.

To achieve exactly-once semantics for the action-audit log in Postgres, we need to ensure that:

Each write operation can be performed multiple times without causing unintended side effects.
If a write fails, it should be possible to roll back and retry without duplicating or overwriting data.
Making DB Writes Idempotent

To make database writes idempotent, we can use strategies such as:

Unique Constraints: Ensure that the actions being logged have unique identifiers (e.g., action_id). If an action with a specific identifier is written multiple times, it should be handled by ensuring only one instance of the write is processed.
Compensating Actions: Define compensating actions for each write operation. For example, if writing a record to Postgres fails, we can implement logic to delete or undo the previous writes in a compensatory transaction.
Versioning and Timestamps: Use version numbers or timestamps to ensure that only the latest version of an action is written.

#### Example
Implementation

Here’s a high-level overview of how we could implement this:

Action Model:

SQL
CREATE TABLE actions (
    id SERIAL PRIMARY KEY,
    action_id UUID NOT NULL UNIQUE,  -- Unique identifier for each action
    action_data JSONB NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

Compensating Logic in the Orchestrator:

PYTHON
def process_action(action):
    try:
        # Attempt to write the action to Postgres
        with db.connection() as conn:
            cursor = conn.cursor()
            cursor.execute("INSERT INTO actions (action_id, action_data) VALUES (%s, %s)",
                           (action['id'], json.dumps(action['data'])))
            conn.commit()
    except psycopg2.IntegrityError:  # Handle unique constraint violation
        conn.rollback()
        logger.warning(f"Action {action['id']} already exists. Skipping.")
    else:
        logger.info(f"Processed action {action['id']} successfully.")

Compensating for Failures:

PYTHON
def compensate_action(action):
    with db.connection() as conn:
        cursor = conn.cursor()
        cursor.execute("DELETE FROM actions WHERE action_id = %s", (action['id'],))
        conn.commit()
        logger.info(f"Compensated for failed action {action['id']}.")

#### Closing Summary
+ Common Mistakes

By ensuring idempotent writes and implementing compensating logic, we can achieve exactly-once semantics in our remediation orchestrator. Common mistakes include:

Failing to handle unique constraints properly.
Not defining or implementing compensatory actions for failed writes.
Overlooking the need for versioning or timestamps.

In summary, while Kafka's transaction guarantees are essential, making database writes idempotent and compensatable is crucial for achieving exactly-once semantics in our action-audit log.

**Comparison:**

```
table "actions" {
    id {type: SERIAL, primary_key: true}
    action_id {type: UUID, unique: true}
    action_data {type: JSONB}
    created_at {type: TIMESTAMP, default: CURRENT_TIMESTAMP}
}
```

**Code:**

```python
import psycopg2

def process_action(action):
    try:
        with db.connection() as conn:
            cursor = conn.cursor()
            cursor.execute("INSERT INTO actions (action_id, action_data) VALUES (%s, %s)",
                           (action['id'], json.dumps(action['data'])))
            conn.commit()
    except psycopg2.IntegrityError:
        conn.rollback()
        logger.warning(f"Action {action['id']} already exists. Skipping.")
    else:
        logger.info(f"Processed action {action['id']} successfully.")

def compensate_action(action):
    with db.connection() as conn:
        cursor = conn.cursor()
        cursor.execute("DELETE FROM actions WHERE action_id = %s", (action['id'],))
        conn.commit()
        logger.info(f"Compensated for failed action {action['id']}.")
```

**Example input / output:**

```text
Input:
action = {'id': '12345678-1234-5678-1234-567890abcdef', 'data': {'status': 'NEW'}}

Output:
Processed action 12345678-1234-5678-1234-567890abcdef successfully.
```

**Data:**

```sql
CREATE TABLE actions (
    id SERIAL PRIMARY KEY,
    action_id UUID NOT NULL UNIQUE,  -- Unique identifier for each action
    action_data JSONB NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Action Received
6. Is Action Id Unique?
7. Write to DB
8. Commit Transaction
9. Log Success
10. Rollback Transaction
11. Log Warning
12. Compensatory Failure
13. Is Compensatory ActionPossible?
14. Delete Failed Action
15. Log Compensation
16. Handle Failure

