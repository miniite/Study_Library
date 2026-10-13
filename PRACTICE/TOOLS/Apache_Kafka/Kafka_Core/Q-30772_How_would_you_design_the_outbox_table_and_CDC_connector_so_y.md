---
type: "practice"
difficulty: "Deep"
---

# How would you design the outbox table and CDC connector so your service never dual-writes and events are never lost even if the app crashes after DB commit?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Quarantine Topics](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Quarantine_Topics.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that your service never dual-writes and events are never lost, especially in scenarios where the application crashes after a database commit, we need to design an outbox table and a CDC (Change Data Capture) connector carefully. The key idea is to establish a reliable mechanism for ensuring that messages are written to Kafka only after they have been successfully committed to the database.

#### Triage

The question revolves around designing a robust system to avoid dual-writes by ensuring that event data is not sent to Kafka until it has been durably stored in the database. This involves creating an outbox table and setting up a CDC connector that can handle various failure scenarios gracefully.

#### Assumptions
The application uses a relational database like PostgreSQL or MySQL.
Kafka is used as the message broker for event-driven systems.
The service is designed to use an outbox pattern, where events are temporarily stored in the database before being sent to Kafka.
Failures can occur during both database commits and Kafka writes.

#### First-Principles Solution

Outbox Table Design:

Create a table that stores event payloads and metadata (e.g., id, event_type, payload, status, created_at, updated_at).
The status column can be used to track the state of each outbox entry (queued, failed, success).

CDC Connector:

Use a CDC library or tool (e.g.,Debezium) that listens for database changes and sends them to Kafka.
Ensure that the CDC connector acknowledges the write operation in the database before sending data to Kafka.

Flow Control Mechanisms:

Implement retry mechanisms for both database commits and Kafka writes.
Use a dead-letter queue (DLQ) to handle messages that cannot be delivered successfully.

#### Cons
istency Checks:

Periodically check the consistency between the outbox table and Kafka topics to ensure no dual-writes or data loss.
Detailed Steps

Outbox Table Schema

SQL
CREATE TABLE outbox (
    id BIGINT PRIMARY KEY,
    event_type VARCHAR(255) NOT NULL,
    payload BYTEA NOT NULL,  -- Use JSONB for complex payloads
    status VARCHAR(50) DEFAULT 'queued',
    created_at TIMESTAMP NOT NULL,
    updated_at TIMESTAMP NOT NULL
);

Inserting Data into Outbox

When an event needs to be persisted, insert a row into the outbox table with status set to 'queued'.

Database Commit and Kafka Write:

SQL
START TRANSACTION;

-- Insert data into outbox
INSERT INTO outbox (id, event_type, payload, created_at) VALUES (...);

-- Attempt to write to Kafka
IF kafka.write(payload) THEN
    UPDATE outbox SET status = 'success', updated_at = CURRENT_TIMESTAMP WHERE id = ...;
ELSE
    UPDATE outbox SET status = 'failed', updated_at = CURRENT_TIMESTAMP WHERE id = ...;
END IF;

COMMIT;

CDC Connector Configuration

Configure the CDC connector to listen for changes and send events to Kafka topics.
Ensure that the connector acknowledges each event after it has been successfully written.

#### Closing Summary

By designing an outbox table and a robust CDC connector, we can ensure that our service never dual-writes and handles failures gracefully. The key is to use database transactions and proper acknowledgment mechanisms to guarantee consistency between the database and Kafka.

#### Common Mistakes/Follow-Ups

Mistake: Forgetting to handle retries properly, leading to data loss.

#### Follow-Up
Implement exponential backoff for retrying failed writes and configure dead-letter queues for handling messages that cannot be delivered successfully.

Mistake: Not checking the consistency between outbox table and Kafka topics periodically.

#### Follow-Up
Schedule periodic health checks or use a monitoring tool to ensure data integrity.

**Comparison:**

```
Column	Data Type	Description
id	BIGINT	Unique identifier for outbox entry
event_type	VARCHAR(255)	Type of the event (e.g., user_registered)
payload	BYTEA	JSONB representation of the event data
status	VARCHAR(50)	State of the outbox entry ('queued', 'failed')
created_at	TIMESTAMP	Timestamp when the entry was created
updated_at	TIMESTAMP	Timestamp when the entry's state was updated
```

**Code:**

```sql
-- Outbox Table Schema
CREATE TABLE outbox (
    id BIGINT PRIMARY KEY,
    event_type VARCHAR(255) NOT NULL,
    payload BYTEA NOT NULL,  -- Use JSONB for complex payloads
    status VARCHAR(50) DEFAULT 'queued',
    created_at TIMESTAMP NOT NULL,
    updated_at TIMESTAMP NOT NULL
);

-- Insert Data into Outbox
INSERT INTO outbox (id, event_type, payload, created_at) VALUES (...);

-- Database Commit and Kafka Write
START TRANSACTION;
INSERT INTO outbox (id, event_type, payload, created_at) VALUES (...);
IF kafka.write(payload) THEN
    UPDATE outbox SET status = 'success', updated_at = CURRENT_TIMESTAMP WHERE id = ...;
ELSE
    UPDATE outbox SET status = 'failed', updated_at = CURRENT_TIMESTAMP WHERE id = ...;
END IF;
COMMIT;
```

**Example input / output:**

```text
Input:
- Event to be processed by the service

Output:
- Inserted into outbox table with `status` set to 'queued'
- Attempted write to Kafka, if successful, update status to 'success', else mark as 'failed'
```

**Data:**

```json
{
  "id": 1,
  "event_type": "user_registered",
  "payload": "{\"username\": \"john_doe\", \"email\": \"john@example.com\"}",
  "status": "queued",
  "created_at": "2023-10-05T14:00:00Z",
  "updated_at": "2023-10-05T14:00:00Z"
}
```

**Flow:**

1. Yes
2. No
3. Event Occurs
4. Database Commit
5. Insert into Outbox Table
6. Kafka Write Successful?
7. Update Outbox Status to'Success'
8. Mark as 'Failed' in Outbox
9. Commit Transaction

