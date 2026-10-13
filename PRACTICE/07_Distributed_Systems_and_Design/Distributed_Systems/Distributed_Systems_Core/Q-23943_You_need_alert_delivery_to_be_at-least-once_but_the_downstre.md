---
type: "practice"
difficulty: "Intermediate"
---

# You need alert delivery to be at-least-once but the downstream ticketing system must not create duplicate tickets on redelivery. Explain the idempotency-key and dedup design on the ticket-creation path and how you handle a redelivery arriving while the first create is in flight.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Idempotent Ingestion](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Idempotent_Ingestion/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

To ensure at-least-once delivery of alerts without causing duplicate tickets, we use an idempotency key to track and deduplicate ticket creations on redelivery. The key ensures that a create request is only processed once, even if it arrives multiple times.

#### Triage

The question probes your understanding of idempotency in distributed systems and how to manage redeliveries without causing duplicates. We'll need to design the system such that each ticket creation operation is idempotent and track its status to prevent duplication during redelivery.

#### Assumptions
The alert delivery service uses a message broker like Kafka or RabbitMQ.
Tickets are created in a downstream ticketing system.
Redeliveries might occur due to network issues, retries, etc.

#### First Principles Solution
Step 1: Message Broker and Delivery Guarantees

The alert messages should be processed at-least-once using mechanisms like Kafka's at-least-once delivery guarantee or an equivalent strategy in RabbitMQ. This ensures that the message is delivered to a consumer even if it fails once.

Producer
Message Broker(Kafka/RabbitMQ)

#### Cons
umer
Step 2: Idempotency Key Generation and Storage

The alert message should include an idempotency key that is unique to each alert. This key will be used to track the creation status of tickets in the ticketing system.

PYTHON
def generate_idempotency_key(alert):
    return hashlib.sha256(json.dumps(alert).encode()).hexdigest()
Step 3: Ticket Creation Logic

The ticket creation logic should check if a ticket with the same idempotency key already exists before creating a new one. This is typically done by querying a database or cache.

PYTHON
def create_ticket_if_not_exists(ticket_key, alert):
    if not ticket_key_exists_in_db(ticket_key):
        # Create and save the ticket in the database
        save_ticket(ticket_key, alert)
Step 4: Handling Redelivery

When a redelivered message arrives while an existing creation request is still in flight, the system should handle it appropriately. This can be achieved by ensuring that only one thread or process handles the creation of tickets for a given idempotency key at any time.

Key Exists
Key Not Found
Redelivery Message
Check Idempotency Key
Ignore Redelivery
Create Ticket, Mark as Created

#### Closing Summary

In summary, to achieve at-least-once delivery without duplicates in the ticketing system, we use an idempotency key to track and deduplicate creation requests. This ensures that even if a message is redelivered, it will only create one new ticket.

#### Common Mistakes
and Follow-Ups:
Failure to Use Idempotency Key: Not generating or checking the idempotency key can lead to duplicate tickets.
Incorrect Handling of Redelivery: Failing to handle redelivery properly can result in unnecessary retries, leading to performance issues.
Database Locking: Not handling concurrent requests correctly might lead to deadlocks or resource contention.

Would you like me to elaborate on any part of this solution?

**Code:**

```python
import hashlib

def generate_idempotency_key(alert):
    return hashlib.sha256(json.dumps(alert).encode()).hexdigest()

def create_ticket_if_not_exists(ticket_key, alert):
    if not ticket_key_exists_in_db(ticket_key):
        # Create and save the ticket in the database
        save_ticket(ticket_key, alert)
```

**Data:**

```json
{
  "alert": {
    "id": "12345",
    "title": "Critical System Failure",
    "details": "The system is down. Immediate action required."
  }
}
```

