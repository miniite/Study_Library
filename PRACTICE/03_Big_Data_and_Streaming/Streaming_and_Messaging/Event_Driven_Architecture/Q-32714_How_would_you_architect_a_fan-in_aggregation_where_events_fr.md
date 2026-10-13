---
type: "practice"
difficulty: "Expert"
---

# How would you architect a fan-in aggregation where events from 200 shards must be joined into a per-customer view, tolerating a shard being hours behind the others?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Idempotent Consumers](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Fat_vs_Thin_Events/Idempotent_Consumers.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To architect an efficient and resilient fan-in aggregation system that handles events from 200 shards and joins them into a per-customer view while tolerating up to several hours of latency between shards, we need to carefully consider several key aspects: message queue design, event processing workflow, data consistency strategies, and fault tolerance mechanisms. This solution balances performance, scalability, and reliability.

The core idea is to use a combination of distributed message queues (e.g., Kafka or RabbitMQ) and an event-driven architecture that ensures eventual consistency between shards while maintaining real-time responsiveness where necessary.

**Comparison:**

```
Shard	Kafka Topic	Aggregator Service	Customer View DB
1	customer_events	aggregator.example.com:8080	postgres://user:password@host:5432/database
...	...	...	...
200	customer_events	aggregator.example.com:8080	postgres://user:password@host:5432/database
```

**Code:**

```
MERMAID
Copy
graph TD;
    A[Shard 1] -->|Event| B[Kafka Topic]
    C[Shard 200] -->|Event| B
    B --> D[Aggregator Service]
    D --> E[Customer View Database]
```

**Example input / output:**

```

An example event flow would be as follows:

Input:

Shard 1 sends an event to Kafka Topic customer_events with the payload { customer_id: '123', event_type: 'login', timestamp: '2023-10-04T16:00Z' }.

Output:

The Aggregator Service processes the event and updates the Customer View Database.
JSON
Copy
{
  "customer_id": "123",
  "event_types": ["login"],
  "latest_timestamp": "2023-10-04T16:00Z"
}
```

**Data:**

```yaml
shards:
  - id: 1
  - id: 200

kafka_topic:
  name: customer_events

aggregator_service:
  host: aggregator.example.com
  port: 8080

customer_view_db:
  db_type: postgresql
  connection_string: "postgres://user:password@host:5432/database"
```

**Flow:**

1. Yes
2. No
3. Shard
4. Event Present?
5. Kafka Producers
6. Kafka Topic
7. Aggregator Service (KafkaConsumer)
8. Customer View DB Update
9. Skip Shard

