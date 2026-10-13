---
type: "practice"
difficulty: "Deep"
---

# How would you design deduplication for an ingestion pipeline writing events to MySQL where the source occasionally replays messages?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Read Your Writes in MySQL](<../../../../TOOLS/MySQL/MySQL_Platform/Read_Your_Writes_in_MySQL.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview

Deduplicating events in a MySQL-based ingestion pipeline, especially when dealing with occasional message replay from sources like Kafka or another queue system, involves designing an efficient and scalable mechanism to ensure that duplicate records are filtered out before they reach the database. The core idea is to leverage unique constraints, indexes, and possibly a combination of application-level logic and database operations.

#### Triaging the Question

The question asks us to design a deduplication strategy for events being written into MySQL from an ingestion pipeline where there might be occasional message replay. Key aspects to consider include:

Ensuring no duplicate records are inserted.
Handling message replay without causing conflicts or performance issues.
Balancing between application-level logic and database constraints.

#### Assumptions
and Solution:
Application-Level Deduplication: A unique key can be added as part of the event payload, ensuring each event is unique.
Database Constraints: Using a primary key or unique index to enforce data integrity.
Message Replay Handling: Ensuring replayed messages are detected and handled without causing conflicts.
Step-by-Step Solution:
1. Application-Level Deduplication

We can add a message_id field to the event payload, which is generated uniquely for each message. This ensures that even if a message is replayed, it will be treated as a new event due to its unique identifier.

SQL
-- Example table schema
CREATE TABLE events (
    id INT AUTO_INCREMENT PRIMARY KEY,
    message_id VARCHAR(255) NOT NULL UNIQUE,  -- Unique identifier for the message
    data TEXT NOT NULL,                       -- Event data
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
2. Database Constraints

To ensure that duplicate messages are not inserted into the database, we use a unique constraint on message_id.

SQL
ALTER TABLE events ADD CONSTRAINT uc_message_id UNIQUE (message_id);

This ensures that if an event with a specific message_id is already present in the table, any subsequent attempt to insert it will fail.

3. Handling Message Replay

For handling message replay, we can implement a retry mechanism at the application level:

Backoff Mechanism: Introduce exponential backoff between retries.
Rate Limiting: Use rate limiting to prevent overwhelming the database with duplicate inserts.
Acknowledge Messages: Ensure that messages are acknowledged after successful insertion into the database.
4. Performance Considerations
Indexing: Create an index on message_id to speed up lookups and ensure efficient unique checks.
Concurrency Control: Use transaction isolation levels (e.g., REPEATABLE READ) to handle concurrent insertions safely.

#### Closing Summary

The solution involves adding a unique identifier (message_id) at the application level, enforcing it as a database constraint, and handling message replay through application logic. This ensures that even if messages are occasionally replayed, duplicate records will be filtered out effectively.

#### Common Mistakes
and Follow-Ups:
Missing Unique Identifier: Not generating or adding a unique identifier for each event.
Overlooking Database Constraints: Relying solely on application-level checks without database support.
Ignoring Message Replay: Failing to implement mechanisms to handle message replay, which can lead to unnecessary retries and performance issues.

For follow-ups, we might discuss:

Scaling: How the solution scales with increasing throughput.
Fault Tolerance: Ensuring robustness against transient failures.
Data Integrity: Additional measures to ensure data integrity beyond just deduplication.

**Comparison:**

| Column | Type          | Constraints            |
|--------|---------------|------------------------|
| id     | INT           | PRIMARY KEY, AUTO_INCREMENT |
| message_id | VARCHAR(255)  | NOT NULL UNIQUE        |
| data   | TEXT          | NOT NULL               |
| created_at | TIMESTAMP    | DEFAULT CURRENT_TIMESTAMP |

**Code:**

```sql
-- Table schema creation
CREATE TABLE events (
    id INT AUTO_INCREMENT PRIMARY KEY,
    message_id VARCHAR(255) NOT NULL UNIQUE,  -- Unique identifier for the message
    data TEXT NOT NULL,                       -- Event data
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Adding unique constraint
ALTER TABLE events ADD CONSTRAINT uc_message_id UNIQUE (message_id);
```

**Example input / output:**

```text
Input:
- Event with message_id = "12345" and data = "Event 1 data"
- Event with message_id = "67890" and data = "Event 2 data"

Output:
- Successfully inserted event with message_id = "12345" (no failure)
- Successfully inserted event with message_id = "67890" (no failure)

Input:
- Event with message_id = "12345" and data = "Duplicate event"

Output:
- Insertion fails due to unique constraint violation
```

**Data:**

```json
{
    "events": [
        {"message_id": "12345", "data": "Event 1 data"},
        {"message_id": "67890", "data": "Event 2 data"},
        {"message_id": "12345", "data": "Duplicate event"}  // This should fail
    ]
}
```

**Flow:**

1. Yes
2. No
3. Receive Event
4. Is Message ID Unique?
5. Insert Event
6. Log Duplicate, Do Not Insert
7. Acknowledge Insertion
8. Retry or Log

