---
type: "practice"
difficulty: "Deep"
---

# How would you architect a saga or outbox pattern on CockroachDB for cross-service transactions, and why can't you just rely on distributed 2PC across microservices even though the DB itself is distributed?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ NewSQL Evaluation and Resilience](<../../../../01_Databases_and_SQL/NoSQL_and_NewSQL/NewSQL/NewSQL_Evaluation_and_Resilience.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To architect a saga or outbox pattern on CockroachDB for handling cross-service transactions, we need to understand that while CockroachDB offers strong consistency and scalability, it doesn't natively support sagas or outbox patterns as these are complex distributed coordination problems. We'll explore how to implement such patterns using CockroachDB's capabilities while highlighting why relying solely on 2PC across microservices is impractical.

#### Triaging the Question

The core of this question is about understanding the limitations and alternatives for managing cross-service transactions in a highly available, distributed system like CockroachDB. We need to explain how sagas or outbox patterns can be implemented and why traditional 2-phase commit (2PC) is not suitable for these use cases.

Why Can't You Rely on Distributed 2PC?
High Latency: 2PC inherently introduces high latency due to the need for coordination between multiple services. This is especially problematic in distributed systems where network delays can be significant.
Transaction Deadlocks: 2PC can lead to deadlocks if multiple transactions are waiting on each other to commit or abort, which can result in a system that appears to hang indefinitely.
Atomicity vs. Availability Trade-off (CAP Theorem): In distributed systems, achieving strong consistency through 2PC often means sacrificing availability in the presence of network partitions. CockroachDB aims for high availability and relaxed consistency models.
Complexity: Implementing 2PC correctly is complex and error-prone, requiring additional infrastructure to handle timeouts, retries, and failure scenarios.
Implementing Saga or Outbox Patterns with CockroachDB

Outbox Pattern:

Definition: The outbox pattern involves batching messages in a local database (like CockroachDB) until all services acknowledge their successful processing.
Workflow:
When a transaction is initiated, the originating service writes an event to its local outbox table.
The service then processes the transaction and updates its state locally.
Once all local operations are completed, the service sends the pending events from the outbox table to the other services.
Each target service processes these events and acknowledges successful processing by updating a confirmation table.
Upon receiving confirmations, the originating service can safely remove the transaction record from its outbox.

Saga Pattern:

Definition: A saga is a sequence of compensating actions that ensure consistency even if one part of the transaction fails.
Workflow:
When initiating a saga, each step in the process writes an atomic transaction.
If a step fails, the compensation steps (which are predefined) are executed to reverse any committed changes.
A global consensus mechanism is used to ensure that all services agree on the rollback.

#### Implementation
Steps with CockroachDB

Create Tables:

SQL
CREATE TABLE outbox (
  id SERIAL PRIMARY KEY,
  event_json JSONB NOT NULL,
  service_id VARCHAR(255) NOT NULL,
  status VARCHAR(50) DEFAULT 'PENDING',
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE confirmations (
  id SERIAL PRIMARY KEY,
  outbox_id INTEGER REFERENCES outbox(id),
  service_id VARCHAR(255) NOT NULL,
  status VARCHAR(50) DEFAULT 'PENDING',
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

Handling Transactions:

In the originating service, use CockroachDB's ACID properties to ensure local transactions are atomic.
Write outbox events and mark them as pending.

Sending Events:

Use an event-driven architecture (like Kafka) or a queue mechanism to send events from the outbox table to target services.
Each target service processes the event and updates its state, then confirms back to the originating service via the confirmation table.

Compensating Actions:

Define compensation actions for each step in the saga sequence.
Use CockroachDB's transactional guarantees to ensure that if a step fails, subsequent steps can be rolled back or retried as needed.

#### Closing Summary
and Common Mistakes

#### Common Mistakes
Failing to handle retries properly, not considering race conditions, neglecting proper error handling, and misunderstanding the implications of distributed consensus mechanisms.
Key Takeaways: While CockroachDB provides strong consistency and scalability, it's crucial to design patterns like sagas or outbox methods carefully to manage cross-service transactions effectively.

**Comparison:**

```
Yes
No
Yes
No
Outbox
Is event pending?
Update confirmation
Remove from outbox
Retry or handle failure
Confirmations
Is confirmation pending?
Mark as completed
I
Handle success or failure
```

**Code:**

```sql
CREATE TABLE outbox (
  id SERIAL PRIMARY KEY,
  event_json JSONB NOT NULL,
  service_id VARCHAR(255) NOT NULL,
  status VARCHAR(50) DEFAULT 'PENDING',
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE confirmations (
  id SERIAL PRIMARY KEY,
  outbox_id INTEGER REFERENCES outbox(id),
  service_id VARCHAR(255) NOT NULL,
  status VARCHAR(50) DEFAULT 'PENDING',
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```

**Example input / output:**

```text
-- Inserting an outbox event
INSERT INTO outbox (event_json, service_id) VALUES ('{"type":"purchase","amount":200}', 'ecommerce');

-- Sending a confirmation back to the originating service
UPDATE confirmations SET status = 'COMPLETED' WHERE outbox_id = 1 AND service_id = 'payment';
```

**Data:**

```json
{
  "outbox": [
    {
      "id": 1,
      "event_json": "{\"type\":\"purchase\",\"amount\":200}",
      "service_id": "ecommerce",
      "status": "PENDING",
      "created_at": "2023-10-05T14:00:00Z"
    }
  ],
  "confirmations": [
    {
      "id": 1,
      "outbox_id": 1,
      "service_id": "payment",
      "status": "PENDING",
      "created_at": "2023-10-05T14:01:00Z"
    }
  ]
}
```

**Flow:**

1. Orig Service
2. CockroachDB Outbox Table
3. Initiate Transaction
4. Local Ops Complete
5. Write to Outbox
6. Send Events via Queue
7. Tar Services Process Events
8. Confirm Success
9. CockroachDB ConfirmationsTable
10. Mark as Completed in Outbox

