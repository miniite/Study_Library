---
type: "practice"
difficulty: "Expert"
---

# Design exactly-once semantics for a message pipeline that consumes from Kafka and writes to Postgres — how do you use transactional offsets or the transactional-outbox to avoid duplicates and lost messages?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Distributed Cache Design](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Distributed_Cache_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary

In designing an exactly-once semantics system for a message pipeline that consumes from Kafka and writes to Postgres, we need to ensure that every message is processed exactly once without duplicates or loss. This involves using transactional offsets in Kafka and the outbox pattern in PostgreSQL to manage the state and commit the transactions only after all processing steps have been successfully completed.

#### Triaging the Question

The core of this question revolves around ensuring reliable, exactly-once delivery semantics for a message pipeline. The interviewer is probing your understanding of distributed systems, transaction management, and how to handle idempotent operations in a scalable manner.

#### Assumptions
We are using Kafka as the messaging system.
Postgres is used for storing application data.
The goal is to ensure that messages are processed exactly once without duplicates or loss.
The pipeline involves multiple steps: consuming from Kafka, processing the message, and persisting the result in Postgres.

#### First Principles Solution
Kafka Consumer Group: Use a single consumer group to process messages from Kafka. This ensures that each message is only consumed by one worker at any given time.
Transaction Management:
Offset Committing: Before committing offsets, ensure all downstream processing steps (e.g., database writes) are successful using transactions.
Idempotent Operations: Use idempotent operations in Postgres to avoid redundant writes.
Outbox Pattern:
Application Layer Outbox: Implement an outbox table in the application layer to track messages that need to be written to Postgres.
Database Transactional Processing: Use database transactions to ensure atomicity, consistency, isolation, and durability (ACID) properties.
Step-by-Step Solution

Kafka Consumer Configuration:

Configure Kafka consumers to use a single consumer group for processing messages.
Ensure that each message is consumed by one worker at a time using fetch.min.bytes and max.partition.fetch.bytes settings.
Use enable.auto.commit = false to disable automatic offset commits.

Transaction Management:

Before committing offsets, wrap the Kafka consumer code in a transactional block.
Ensure that all operations (e.g., fetching messages, processing, and writing to Postgres) are part of this transaction.
Use BEGIN TRANSACTION; before starting the processing and commit or rollback based on success.

Outbox Pattern:

Create an outbox table in Postgres to track messages that need to be written.
SQL
CREATE TABLE outbox (
    id SERIAL PRIMARY KEY,
    message_id UUID NOT NULL,
    event_type VARCHAR(255) NOT NULL,
    payload JSONB NOT NULL,
    processed BOOLEAN DEFAULT FALSE
);
When a new message is received, insert its details into the outbox table.
Use an UPSERT (INSERT or UPDATE) statement to handle idempotent writes in Postgres:
SQL
INSERT INTO outbox (message_id, event_type, payload)
VALUES ($1, $2, $3)
ON CONFLICT (message_id) DO NOTHING;
After successfully processing the message and inserting it into the database, update the processed field to TRUE.

Error Handling:

Use retries and dead-letter queues for handling failures.
Implement backoff strategies for retrying failed transactions.

Monitoring and Logging:

Monitor offset commits and outbox table updates for any anomalies.
Log all critical operations for debugging purposes.

#### Closing Summary

In summary, achieving exactly-once semantics in a message pipeline involves using Kafka consumer groups to ensure exclusive processing of messages, transactional management for database writes, and the outbox pattern to handle idempotent writes. This approach ensures that each message is processed only once, avoiding duplicates and ensuring no data loss.

#### Common Mistakes/Follow-Ups
Mistake 1: Failing to use transactional blocks around database operations.
Mistake 2: Not using the outbox pattern for idempotent writes, leading to redundant messages in Postgres.

#### Follow-Up
Discussing how to handle retries and dead-letter queues, and implementing proper error handling mechanisms.

**Comparison:**

```
Outbox	Description
id	Unique ID
message_id	Message identifier
event_type	Type of event
payload	JSONB representation of the message
```

**Code:**

```python
# Example Kafka Consumer Code with Transaction Management
from kafka import KafkaConsumer
import psycopg2

consumer = KafkaConsumer('topic_name', group_id='group_id', auto_offset_reset='earliest')
conn = psycopg2.connect(database="dbname", user="user", password="pass", host="localhost", port="5432")
cursor = conn.cursor()

for message in consumer:
    with conn.transaction():
        try:
            # Process the message
            processed_message = process_message(message.value)

            # Upsert into outbox table
            cursor.execute("INSERT INTO outbox (message_id, event_type, payload) VALUES (%s, %s, %s) ON CONFLICT (message_id) DO NOTHING",
                           (processed_message['id'], 'event_type', processed_message['payload']))

            conn.commit()
        except Exception as e:
            print(f"Error processing message: {e}")
            conn.rollback()

# Ensure all connections are closed
cursor.close()
conn.close()
```

**Example input / output:**

```text
Example input to output flow:
- Input Message: `{"id": "1", "event_type": "create_user"}` (Kafka message)
- Output Table Update: Insert into outbox table with `message_id`, `event_type`, and `payload`.
```

**Data:**

```sql
CREATE TABLE outbox (
    id SERIAL PRIMARY KEY,
    message_id UUID NOT NULL,
    event_type VARCHAR(255) NOT NULL,
    payload JSONB NOT NULL,
    processed BOOLEAN DEFAULT FALSE
);
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Receive Kafka Message
6. Is Outbox Entry Present?
7. Mark as Processed
8. Log Success
9. Insert into Outbox Table
10. Process Message
11. Commit Transaction
12. Mark as Processed in Outbox
13. Retry on Failure?

