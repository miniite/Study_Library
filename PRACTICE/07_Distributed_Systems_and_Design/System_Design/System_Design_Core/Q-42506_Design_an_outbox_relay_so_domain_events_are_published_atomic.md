---
type: "practice"
difficulty: "Deep"
---

# Design an outbox + relay so domain events are published atomically with the DB commit. How does the relay guarantee at-least-once to Kafka, and how do downstream consumers dedupe?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Multi-tenant Migrations](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Read-through_and_Write-behind_Caching/Multi-tenant_Migrations.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a system that ensures atomicity between database commits and event publishing while guaranteeing at least once delivery on Kafka and handling deduplication in downstream consumers, we need to break down the problem into several key components:

Outbox Mechanism: We will use an outbox pattern to queue events before they are published.
Relay Layer: A component that handles the coordination between the database commit and event publishing.
Kafka Producer: Ensures at least once delivery on Kafka.
Downstream Consumers: Implement deduplication logic.
TL;DR

We will implement an outbox mechanism to queue events before they are published, use a relay layer to ensure atomicity with the database commit, publish these events atomically using a reliable Kafka producer, and handle deduplication in downstream consumers through unique identifiers or timestamps.

#### Triaging the Question

The question is asking for a system design that ensures:

Atomicity: Events are published only after their corresponding transactions have been committed.
At-least-once Delivery on Kafka: Ensures all events are delivered at least once to the Kafka topic.
Deduplication in Downstream Consumers: Prevents duplicate processing of the same event.

#### Assumptions
The database transaction and outbox event storage use a reliable system (e.g., ACID properties).
Kafka can be configured for at-least-once delivery using idempotent producers.
Downstream consumers have some form of unique identifier or timestamp to detect duplicates.

#### First-Principles Solution
1. Outbox Mechanism

We will implement an outbox pattern where events are stored in a separate table before they are published. This ensures that the event is not lost if the publishing process fails.

SQL
CREATE TABLE EventOutbox (
    Id BIGINT PRIMARY KEY,
    DomainEntityId BIGINT NOT NULL,
    JsonData TEXT NOT NULL,
    Published BOOLEAN DEFAULT FALSE
);
2. Relay Layer (Reliable Transaction Manager)

The relay layer will be responsible for ensuring that the event is only published after the database commit has been confirmed.

PYTHON
class OutboxRelay:
    def __init__(self, db_connection, kafka_producer):
        self.db = db_connection
        self.kafka = kafka_producer

    def handle_transaction(self, domain_entity_id, json_data):
        with self.db.atomic():
            # Insert the event into the outbox table
            outbox_event = EventOutbox(
                DomainEntityId=domain_entity_id,
                JsonData=json_data,
                Published=False
            )
            outbox_event.save()

            # Publish the event to Kafka
            kafka_key = f"event_{outbox_event.Id}"
            self.kafka.send('events_topic', key=kafka_key, value=outbox_data)
            outbox_event.Published = True
            outbox_event.save()
3. Kafka Producer Configuration

To ensure at-least-once delivery, we configure the Kafka producer to use idempotent mode.

PYTHON
from kafka import KafkaProducer

producer = KafkaProducer(bootstrap_servers='localhost:9092', enable_idempotence=True)
4. Downstream Consumers (Deduplication Logic)

Downstream consumers can implement deduplication logic based on unique identifiers or timestamps.

PYTHON
class EventConsumer:
    def __init__(self, kafka_consumer):
        self.consumer = kafka_consumer

    def consume(self):
        for message in self.consumer.poll():
            key = message.key.decode()
            value = message.value.decode()

            # Check if the event has already been processed
            if not self.is_duplicate(key, value):
                process_event(value)

    def is_duplicate(self, key, value):
        # Example: check based on a unique identifier or timestamp
        return EventOutbox.get_or_none(EventOutbox.Id == int(key.split('_')[-1]))

#### Closing Summary
and Common Mistakes/Follow-Ups

#### Closing Summary
The design ensures atomicity by using an outbox mechanism, guarantees at-least-once delivery on Kafka by configuring the producer properly, and prevents duplicate processing in downstream consumers through deduplication logic. The key is to use a reliable database connection for transactions, configure idempotent producers for Kafka, and implement robust deduplication checks.

#### Common Mistakes/Follow-Ups

Losing Events: Ensure that the outbox mechanism does not lose events if the publishing process fails.
Duplicate Processing: Implementing effective deduplication logic in downstream consumers to avoid reprocessing of identical events.
Error Handling: Robust error handling and retries mechanisms should be implemented both for database transactions and Kafka messages.

**Comparison:**

| Id | DomainEntityId | JsonData                          | Published |
|----|---------------|----------------------------------|-----------|
| 1  | 12345         | {'event_type': 'user_registered'} | False     |
| 2  | 12346         | {'event_type': 'order_placed'}    | True      |

**Code:**

```

class OutboxRelay: def init(self, db_connection, kafka_producer): self.db = db_connection self.kafka = kafka_producer

CODE
Copy
def handle_transaction(self, domain_entity_id, json_data):
    with self.db.atomic():
        outbox_event = EventOutbox(
            DomainEntityId=domain_entity_id,
            JsonData=json_data,
            Published=False
        )
        outbox_event.save()

        kafka_key = f"event_{outbox_event.Id}"
        self.kafka.send('events_topic', key=kafka_key, value=outbox_data)
        outbox_event.Published = True
        outbox_event.save()
CODE
Copy

```python
class EventConsumer:
    def __init__(self, kafka_consumer):
        self.consumer = kafka_consumer

    def consume(self):
        for message in self.consumer.poll():
            key = message.key.decode()
            value = message.value.decode()

            if not self.is_duplicate(key, value):
                process_event(value)

    def is_duplicate(self, key, value):
        return EventOutbox.get_or_none(EventOutbox.Id == int(key.split('_')[-1]))
```

**Example input / output:**

```text
# Example Input-Output for the relay handling transaction and publishing events

def handle_transaction(domain_entity_id, json_data):
    outbox_relay = OutboxRelay(db_connection, kafka_producer)
    outbox_relay.handle_transaction(domain_entity_id, json_data)

handle_transaction(12345, "{'event_type': 'user_registered', 'data': {}}")
```

**Data:**

```sql
CREATE TABLE EventOutbox (
    Id BIGINT PRIMARY KEY,
    DomainEntityId BIGINT NOT NULL,
    JsonData TEXT NOT NULL,
    Published BOOLEAN DEFAULT FALSE
);
```

**Flow:**

1. Transaction Starts
2. Database Commit
3. Outbox Event Inserted
4. Kafka Message Sent
5. Mark Outbox as Published
6. Deduplicate Check inConsumer
7. Process Event if Not Duplicate
8. Consumer AcknowledgesProcessing

