---
type: "practice"
difficulty: "Expert"
---

# How do you architect data governance and lineage across an event backbone so that when a regulator asks 'where did this customer's data flow', you can answer across every topic and consumer without instrumenting each team by hand?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Breaking Schema Changes](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Fat_vs_Thin_Events/Breaking_Schema_Changes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To architect data governance and lineage in an event-driven system, we need to ensure that the system can provide a clear and unambiguous trace of data flows through various topics and consumers. The key challenge is to do this without needing to instrument each team individually, which would be time-consuming and error-prone.

In essence, the interviewer is probing how you design a robust, scalable, and maintainable system for tracking data lineage. This involves understanding the architecture principles, designing an event backbone that supports tracing, and implementing mechanisms for recording and retrieving this information efficiently.

#### Assumptions

The system uses a message queue infrastructure (e.g., Kafka).
Each topic in the event backbone is a logical data stream.

#### Cons
umers are systems or processes consuming these streams.

Step-by-step solution from first principles:

Event Backbone Design: Use a robust, scalable message queue like Apache Kafka to handle events. Kafka’s distributed nature and support for multiple consumers make it ideal for this scenario.

Metadata Infrastructure: Implement a metadata store (e.g., a relational database) that records details about each event as it flows through the system. This metadata includes timestamps, topics, consumer identifiers, and any relevant attributes.

Event Enrichment: As events are produced, enrich them with context from various sources such as user attributes, timestamp, source application, etc. Store this enriched data alongside the raw event in the message queue.

Lineage Tracking Mechanism: Develop a lineage tracking mechanism that logs every transformation and movement of an event. This could be done by creating a sidecar component or using Kafka Streams to process each incoming event and record its journey.

#### Cons
umer Layer Instrumentation: While we aim to avoid instrumenting each consumer, the consumers themselves must still report their actions (e.g., when they start processing an event). This can be achieved through lightweight logging or by implementing a middleware layer that processes events before distribution.

Data Aggregation and Querying: Design an aggregation mechanism where periodic snapshots of lineage data are stored in a searchable format, allowing for quick queries to track historical data flows.

Audit Logs and Alerts: Implement audit logs and alerts to notify when certain events or patterns arise that might indicate potential compliance issues.

#### Closing summary
By leveraging Kafka’s event streaming capabilities and complementing them with a robust metadata and lineage tracking system, we can ensure complete visibility into the flow of customer data without overburdening individual teams. This approach scales well and provides the necessary insights to regulators while maintaining operational efficiency.

#### Common mistakes/follow-ups

Overcomplicating the Solution: Avoid designing overly complex solutions that might be difficult to maintain.
Neglecting Performance Considerations: Ensure that the metadata tracking does not introduce significant latency or performance issues in the main event processing pipeline.
Failing to Incorporate Feedback Loops: The system should allow for feedback from regulatory bodies and internal audits to continuously improve data governance practices.

**Comparison:**

```
Event ID	Topic	Consumer Identifier	Timestamp	Attributes
12345	user_registration	customer_service_team	2023-10-01T12:00:00Z	{user_id: 98765}
```

**Code:**

```
MERMAID
Copy
graph TD;
    A[Event Produced] --> B[Kafka Topic]
    B --> C{Is Event Enriched?}
    C -- Yes --> D[Metadata Store]
    C -- No --> E[Sidecar Processor]
    E --> F[Enriched Event + Metadata] --> G[Audit Log]
```

**Example input / output:**

```

Input (raw event):

JSON
Copy
{
  "event_type": "user_registration",
  "user_id": 98765,
  "timestamp": "2023-10-01T12:00:00Z"
}

Output (enriched and logged event):

JSON
Copy
{
  "event_id": 12345,
  "topic": "user_registration",
  "consumer_identifier": "customer_service_team",
  "timestamp": "2023-10-01T12:00:00Z",
  "attributes": {
    "user_id": 98765
  }
}
```

**Data:**

```yaml
metadata_store:
  - event_id: 12345
    topic: user_registration
    consumer_identifier: customer_service_team
    timestamp: 2023-10-01T12:00:00Z
    attributes:
      user_id: 98765
```

**Flow:**

1. Yes
2. No
3. Event Produced
4. Kafka Topic
5. Is Event Enriched?
6. Metadata Store
7. Sidecar Processor
8. Enriched Event + Metadata
9. Audit Log

