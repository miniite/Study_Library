---
type: "practice"
difficulty: "Deep"
---

# Your team dual-writes events to both the OLTP DB and Kafka, and they occasionally disagree. Explain the outbox pattern and how it would fix your reliability problem.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Real-time OLAP Serving Layer](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/Real-time_OLAP_Serving_Layer.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview

In a typical data architecture where transactions are simultaneously written to an operational database (OLTP) and a messaging system like Kafka, discrepancies can arise due to eventual consistency issues. The outbox pattern is designed to address this problem by ensuring that writes are committed exactly once. By implementing the outbox pattern, we can ensure that all changes made in the OLTP database are also recorded in an internal audit or event store, which can then be replayed if discrepancies occur.

#### Triage

The question revolves around understanding and explaining how the outbox pattern can resolve the issue of conflicting writes between a transactional database (OLTP) and a messaging system like Kafka. The key points to consider are:

Transaction Synchronization: Ensuring that all changes in the OLTP database are propagated consistently.
Event Propagation: Mechanisms for reliably forwarding events from the OLTP to external systems such as Kafka.
Problem Statement

The outbox pattern addresses a scenario where transactions are dual-writes to both an OLTP database and a messaging system. Occasionally, these writes can fall out of sync due to various reasons like network failures or inconsistent transaction processing times. The goal is to guarantee that all changes made in the OLTP system are recorded exactly once and consistently across different systems.

#### Assumptions
Eventual Consistency: There might be a delay between when an event is committed in the OLTP database and when it appears in Kafka.
Transaction Logging: The OLTP database maintains a log of all changes made by transactions, which can later be replayed if needed.

#### First-Principles Solution
Step 1: Event Capture

When a transaction modifies data in the OLTP database, an outbox entry is created to record this change. This entry contains metadata such as the timestamp and unique identifier for the event.

SQL
CREATE TABLE outbox (
    id INT PRIMARY KEY,
    message_type VARCHAR(255),
    payload JSONB,
    processed_at TIMESTAMP NULL,
    last_attempted_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    retry_count INT NOT NULL DEFAULT 0
);
Step 2: Event Propagation

After a transaction commits, the outbox entry is marked as pending for propagation. A background job or scheduled task picks up these entries and sends them to Kafka.

JAVA
// Pseudocode
if (outbox_entry.processed_at IS NULL) {
    sendToKafka(outbox_entry);
}
Step 3: Event Acknowledgment

Once an outbox entry is successfully written to Kafka, it is marked as processed. If the write to Kafka fails, the entry is retried after a certain backoff strategy.

JAVA
if (sendToKafka(outbox_entry)) {
    updateProcessedStatus(outbox_entry.id);
}
Step 4: Event Replay

If there are discrepancies between the OLTP database and Kafka, the outbox entries can be replayed. This ensures that all missed events or erroneous writes are corrected.

SQL
UPDATE outbox SET processed_at = CURRENT_TIMESTAMP WHERE id IN (SELECT id FROM outbox WHERE processed_at IS NULL);

#### Closing Summary

The outbox pattern provides a robust solution to ensure that dual-writes between an OLTP database and Kafka maintain eventual consistency. By capturing, propagating, and replaying events through this mechanism, we can handle failures and discrepancies in a controlled manner.

#### Common Mistakes/Follow-Ups
Incomplete Propagation: Failing to acknowledge successful writes to Kafka or not properly handling retries.
Synchronization Delays: Not considering the potential delays between OLTP commits and Kafka acknowledgments.
Scalability Concerns: Understanding how this pattern scales with large volumes of events and complex transaction workflows.

In summary, implementing the outbox pattern is a practical approach to maintaining data integrity across multiple systems in a modern microservices architecture.

**Comparison:**

```
tablechart
row id, message_type, payload, processed_at, last_attempted_at, retry_count
1, USER_UPDATED, {"user_id": 123, "name": "John Doe"}, null, 2023-10-01T10:00:00Z, 0
```

**Code:**

```sql
CREATE TABLE outbox (
    id INT PRIMARY KEY,
    message_type VARCHAR(255),
    payload JSONB,
    processed_at TIMESTAMP NULL,
    last_attempted_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    retry_count INT NOT NULL DEFAULT 0
);
```

**Example input / output:**

```text
Input:
- Transaction modifies user data in OLTP database.

Output:
1. Outbox entry created with updated payload and timestamp.
2. Background job picks up the outbox entry and sends it to Kafka.
3. Kafka message sent successfully, outbox entry marked as processed.
4. If Kafka fails, outbox entry is retried after backoff strategy.
5. If discrepancies arise, outbox entries are replayed from OLTP database.
```

**Data:**

```json
{
    "id": 1,
    "message_type": "USER_UPDATED",
    "payload": {"user_id": 123, "name": "John Doe"},
    "processed_at": null,
    "last_attempted_at": "2023-10-01T10:00:00Z",
    "retry_count": 0
}
```

**Flow:**

1. Yes
2. Success
3. Failure
4. Transaction modifies data inOLTP
5. Is outbox entry needed?
6. Create outbox entry
7. Send to Kafka
8. Mark as processed
9. Retry with backoff
10. Replay if necessary

