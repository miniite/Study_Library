---
type: "practice"
difficulty: "Expert"
---

# Architect the event system for a payments platform where every event must be exactly-once against the ledger, auditable for a decade, and resilient to a full region loss. Walk me through the layers and where you accept eventual consistency versus demand strong guarantees.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Idempotent Consumers](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Fat_vs_Thin_Events/Idempotent_Consumers.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To architect an event system for a payments platform that ensures exactly-once delivery against the ledger, is auditable for a decade, and resilient to full region loss, we need to carefully balance between strong consistency and eventual consistency principles. This involves designing multiple layers including the event source layer, message queue layer, processing layer, persistence layer, and audit layer.

The core requirement here is exactly-once delivery, which means every event should be processed only once. Additionally, the system must support auditing for a decade to ensure compliance with regulatory requirements. Finally, it needs resilience against full region loss—meaning that if an entire region fails, the system can continue functioning without losing data or failing to process events.

To meet these requirements, we will employ a layered architecture and make strategic trade-offs where necessary.

Event Source Layer: This layer captures raw events from various sources such as user interactions, API calls, or other systems.
Message Queue Layer: Manages the delivery of messages between different components with guaranteed exactly-once processing.
Processing Layer: Handles event processing logic and updates state in the ledger while ensuring data integrity.
Persistence Layer: Ensures that all processed events are durably stored, supporting auditing requirements.
Audit Layer: Tracks every change and maintains a comprehensive audit trail for regulatory compliance.

Let's dive into each layer:

Event Source Layer

This layer is responsible for ingesting raw events from various sources. For example, when a user initiates a payment, the request reaches this layer. The key challenge here is to ensure that these events are captured reliably and accurately.

Message Queue Layer

This is where we accept eventual consistency as a trade-off. We use a reliable message queue like Amazon SQS or RabbitMQ to handle event delivery between layers. The idea here is to buffer events in the queue, ensuring they are processed exactly once by the system but not necessarily immediately. This layer ensures that even if some components fail temporarily, events will still be processed when the system recovers.

Processing Layer

This layer processes each message from the queue with exact-once semantics using techniques like idempotent commands and compensating transactions. It updates the ledger state atomically to ensure no duplicate or missing operations. We can achieve this by maintaining a unique identifier for each event and checking its status before processing it again.

Persistence Layer

For auditing, we need persistent storage that can survive a full region loss. Using a distributed database like Cassandra or a time-series database like InfluxDB is appropriate here. The key requirement is that all processed events are stored in such a way that they can be audited for a decade. This involves implementing robust backups and recovery mechanisms.

Audit Layer

This layer tracks every change made to the ledger and logs it meticulously. For regulatory compliance, we need to ensure that audit trails are preserved securely and cannot be tampered with. Techniques like log shipping or multi-master replication can help maintain consistency across multiple nodes while ensuring data integrity.

#### Trade-offs
and Considerations
Exactly-once vs. Eventual Consistency: We accept eventual consistency in the message queue layer but demand strong guarantees elsewhere, particularly in the processing and persistence layers.
Scalability and Performance: The system must be designed to handle high volumes of events efficiently while maintaining reliability and performance.
Cost and Complexity: Implementing such a robust architecture comes with added costs related to infrastructure, storage, and maintenance. However, it is crucial for compliance and business continuity.

#### Conclusion

By architecting the event system in layers, we can balance between strong consistency requirements and practical trade-offs. The key is to ensure that critical components like processing and persistence maintain exactly-once guarantees while using message queues where eventual consistency is acceptable. This approach not only meets our functional requirements but also ensures the system remains scalable, resilient, and compliant.

#### Common mistakes
include neglecting audit trails or assuming that a single layer can handle all consistency needs without proper separation of concerns.

**Comparison:**

```
Layer	Functionality	Consistency Model
Event Source	Ingest raw events	Eventually Consistent
Message Queue	Buffer messages	Eventually Consistent
Processing	Apply exactly-once processing logic	Strongly Consistent
Persistence	Store processed data	Strongly Consistent
Audit	Maintain audit trails	Strongly Consistent
```

**Code:**

```python
# Example code for exactly-once processing in the Processing Layer
def process_event(event_id):
    if not check_event_status(event_id):  # Check if event is already processed
        try:
            apply_event_to_ledger(event_id)  # Apply changes to the ledger
            mark_event_as_processed(event_id)  # Mark event as processed
        except Exception as e:
            log_error(e)
```

**Example input / output:**

```text
Input:
- Raw event generated from a payment request

Output:
1. Event is captured and buffered in the message queue.
2. Message is processed exactly once using idempotent commands.
3. Ledger state is updated atomically, and event status is marked as processed.
4. Updated ledger state and event details are stored durably for auditing.
```

**Data:**

```json
{
  "event": {
    "id": "1234567890",
    "type": "PaymentInitiated",
    "timestamp": "2023-10-01T12:00:00Z"
  },
  "ledger_state": {
    "balance": -10.50,
    "last_updated": "2023-10-01T12:00:00Z"
  }
}
```

**Flow:**

1. Event Source
2. Check Event Status
3. Apply Event to Ledger
4. Mark as Processed
5. B.negative
6. Log Error
7. Persist Data for Auditing

