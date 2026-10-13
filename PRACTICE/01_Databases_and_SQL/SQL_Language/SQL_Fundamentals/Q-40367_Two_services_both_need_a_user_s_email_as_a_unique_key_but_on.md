---
type: "practice"
difficulty: "Deep"
---

# Two services both need a user's email as a unique key, but one is being extracted into its own DB. How do you preserve uniqueness and referential guarantees once they no longer share a database?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Foreign Keys and Referential Integrity](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Foreign_Keys_and_Referential_Integrity.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the problem of preserving uniqueness and referential integrity between two services that no longer share a database, we need to ensure that each service can independently manage user emails while maintaining consistency across both systems. This involves designing robust mechanisms for data synchronization and validation.

First, let's triage the question. The interviewer is probing how you would maintain data integrity when splitting a database into multiple services, specifically focusing on email uniqueness and referential guarantees. Key assumptions include:

Each service needs to ensure that no duplicate emails exist.
There should be a mechanism for one service (let’s call it Service A) to extract user information and update the other service (Service B).
Both services must have fail-safe mechanisms to handle potential data conflicts or inconsistencies.

From first principles, we can break this down step-by-step:

Data Synchronization: Implement a system where updates in one database (Service A's) trigger an event that is consumed by Service B.
Validation and Conflict Resolution: Ensure both services validate emails independently but also have mechanisms to resolve any conflicts if they arise.

Let’s explore these steps in more detail:

Step 1: Data Synchronization
Mechanism

Use a distributed event-driven architecture where changes in one service can be propagated to another via an event bus or message queue. For example, when Service A updates or deletes a user record, it publishes an event containing the relevant data.

Update/Delete Event

#### Cons
umes Event
Service A
Event Bus
Service B

#### Example
Schema for Event Bus
JSON
{
  "eventType": "USER_UPDATE",
  "userId": "12345",
  "email": "user@example.com"
}
Step 2: Validation and Conflict Resolution
Independent Validation in Each Service

Both services should implement a validation layer to ensure no duplicate emails are inserted. This can be done using unique constraints on the database level.

SQL
CREATE TABLE User (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL
);
Conflict Resolution Mechanism

In case of conflicts, one approach is to use a version number or timestamp in the user record. If Service B detects an existing email during synchronization, it can compare timestamps or version numbers to decide whether to update, ignore, or reject the incoming change.

Step 3: Implementing Synchronization

Here’s a simplified example using Kafka as the event bus:

Service A:

On user creation/update/deletion, publish an event.

Event Bus (Kafka):

#### Cons
ume events from Service A and forward them to Service B.

Service B:

Listen for incoming events, validate the email against its own database, and update if necessary.
Step 4: Handling Edge Cases
Duplicate Emails: Both services should log such conflicts and handle them according to predefined policies (e.g., reject the change, ask for manual intervention).
Latency Issues: Ensure high availability of both event bus and services. Use retries and dead-letter queues to manage failed updates.
Data Consistency: Implement strong consistency strategies if possible, or accept eventual consistency.

#### Closing Summary

By using an event-driven architecture with a reliable message queue, both services can maintain referential integrity through independent validation and conflict resolution mechanisms. This approach ensures that even though the databases are separate, their combined behavior mimics a single database in terms of data consistency and uniqueness guarantees.

#### Common Mistakes/Follow-Ups

Ignoring eventual consistency implications.
Failing to implement proper logging and error handling.
Overlooking potential performance bottlenecks in event processing.

**Code:**

```python
# Example Python code for Service A publishing an event
from kafka import KafkaProducer

producer = KafkaProducer(bootstrap_servers='localhost:9092')
event = {"eventType": "USER_UPDATE", "userId": "12345", "email": "user@example.com"}
producer.send('user_events', value=event)
```

**Example input / output:**

```text
- **Input**: Event from Service A to Kafka event bus.
- **Output**: Update in Service B after validation.
```

**Data:**

```json
{
  "eventType": "USER_DELETE",
  "userId": "67890"
}
```

