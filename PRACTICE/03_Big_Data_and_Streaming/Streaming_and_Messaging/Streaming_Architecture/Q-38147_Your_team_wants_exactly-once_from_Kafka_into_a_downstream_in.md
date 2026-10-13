---
type: "practice"
difficulty: "Deep"
---

# Your team wants exactly-once from Kafka into a downstream incident-management API that isn't transactional. Explain how you'd get effectively-once behavior and what idempotency key you'd derive from the alert.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Consuming Untrusted Streams](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Provisional_and_Final_Results/Consuming_Untrusted_Streams.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To achieve effectively-once processing in real-time streaming from Kafka to a downstream incident-management API that doesn't support transactions, we need to design an architecture that ensures each event is processed exactly once (EON). This involves leveraging idempotent keys and idempotent processing mechanisms. The core idea is to use a combination of message deduplication strategies and consistent state management.

First, let's triage the question:

The interviewer is probing our understanding of EON semantics, how they can be achieved in distributed systems like Kafka, and the nuances of integrating with APIs that lack transactional support.
We need to ensure idempotency at both the messaging layer (Kafka) and the downstream processing layer.

The key assumptions are:

The incident-management API is stateless or lacks a mechanism for deduplication.
Kafka's semantics do not guarantee exactly-once delivery without proper handling in the consumer logic.
We aim to achieve effectively-once behavior by combining Kafka's at-least-once (ALO) guarantees with idempotent processing.

To get effectively-once behavior, we can use a combination of:

Idempotent Key Generation: Derive a unique key from each alert that is stable and consistent.
State Management: Maintain a state store to track processed messages.

Here's the step-by-step solution:

Step 1: Idempotent Key Generation

We derive an idempotent key from the alert payload. This can be done in multiple ways, but common approaches include:

Using a combination of fields like timestamp and unique identifier.
Generating a hash or message digest based on critical fields.

#### Example

PYTHON
def generate_idempotent_key(alert):
    return f"{alert['timestamp']}-{alert['unique_identifier']}"
Step 2: State Management with Kafka

We use Kafka Streams to maintain state. Each key will be used as the identifier for a record in our state store. Kafka's in-memory storage can be leveraged, or we can integrate with an external state store like Apache BookKeeper.

Alert Producer
Kafka Topic
Kafka Streams Application
State Store
Step 3: Idempotent Processing Logic

In the Kafka Streams application, we implement a custom processing logic that checks if the key has already been processed. If it hasn't, we process the message and update the state store.

JAVA
StreamsBuilder builder = new StreamsBuilder();
KStream<String, String> source = builder.stream("alerts");
source.groupByKey().windowedBy(TimeWindows.of(Duration.ofMinutes(5)))
      .reduce((alert1, alert2) -> {
          // Check if idempotent key has been seen before
          if (!stateStore.contains(alert1.key())) {
              processAlert(alert1.value());
              stateStore.put(alert1.key(), true);
          }
          return alert2;  // Return the latest alert
      });
Step 4: Downstream Integration

The processed alerts are then sent to the incident-management API. We ensure that each message is idempotent by including a unique key in the request headers or URL.

PYTHON
def send_to_api(alert, state_store):
    if not state_store.contains(generate_idempotent_key(alert)):
        # Prepare and send the alert to the API
        response = requests.post("https://api.example.com/incidents", data=alert)
        state_store.put(generate_idempotent_key(alert), True)

#### Closing Summary

By combining Kafka's ALO guarantees with idempotent processing logic, we can achieve effectively-once behavior in our system. The key steps include generating a stable idempotent key and maintaining state to ensure that each message is processed exactly once.

#### Common Mistakes/Follow-Ups

Not properly handling the idempotent keys, leading to duplicate processing.
Failing to maintain consistent state across nodes if using an external store.
Ignoring the potential overhead of state management in high-throughput scenarios.

**Comparison:**

```
Processing State	Key	Value
Initial	a1	Not Seen
After Processing	a1	Seen
Initial	b2	Not Seen
After Processing	b2	Seen
```

**Example input / output:**

```

Example input:

JSON
Copy
[
    {"timestamp": 1634500800, "unique_identifier": "a1", "event_type": "error"},
    {"timestamp": 1634500801, "unique_identifier": "b2", "event_type": "warning"}
]

Example output:

JSON
Copy
[
    {"timestamp": 1634500800, "unique_identifier": "a1", "processed": true},
    {"timestamp": 1634500801, "unique_identifier": "b2", "processed": true}
]
```

**Flow:**

1. Alert Producer
2. Kafka Topic
3. Kafka Streams Application
4. State Store
5. Incident-Management API

