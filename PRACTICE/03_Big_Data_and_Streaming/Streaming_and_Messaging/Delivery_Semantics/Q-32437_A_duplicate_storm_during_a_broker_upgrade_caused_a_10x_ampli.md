---
type: "practice"
difficulty: "Expert"
---

# A duplicate storm during a broker upgrade caused a 10x amplification because each retry triggered downstream retries. Analyze the retry-amplification failure mode and design backpressure and dedup that break the feedback loop.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Dedup Contracts](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Delivery_Semantics/Dedup_Contracts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary: In this scenario, we are dealing with a critical issue where duplicate storms during a broker upgrade led to an exponential increase in retry attempts, creating a harmful feedback loop. To address this, we need to design a robust backpressure mechanism and deduplication strategy that prevents such amplification.

#### Triage
The question is asking us to analyze the root cause of retry-amplification failure mode during a broker upgrade and design an effective solution involving backpressure and deduplication. We'll make several assumptions:

The system uses message brokers (e.g., Kafka, RabbitMQ).
Retries are configured with exponential backoff.
There is a mechanism for detecting duplicate messages.

#### First-Principles Solution

Understand the Problem: When a broker upgrade introduces instability, some messages might be processed multiple times due to retries. This leads to an exponential increase in retry attempts, causing downstream services to receive excessive requests, potentially leading to a storm and eventual failure.
Backpressure Mechanism: Implement a backpressure mechanism that limits the rate at which messages are sent to consumers based on their current processing capacity. This can be done using flow control protocols like Sliding Windows or Token Bucket algorithms.
Deduplication Strategy: Introduce deduplication strategies to ensure that duplicate messages are not processed multiple times, even if they trigger retries. This can be achieved through message headers, database lookups, or cache mechanisms.
Step-by-Step Solution:

Flow Control and Backpressure:

Sliding Window: Implement a sliding window mechanism where each consumer is allocated a certain number of messages within a time frame (e.g., 5 messages in the last 30 seconds). If this limit is exceeded, further requests are throttled.
Exceeded Limits?
Not Exceeded

#### Cons
umer
Receive Message
Process Message
Send ACK to Broker
Check Sliding Window
Backpressure (ThrottleRequests)
Continue Processing

Deduplication Strategy:

Message Headers: Add a unique identifier in the message headers that is checked before processing.
Database Lookups: Store processed messages in a database and check for duplicates using an index or hash of the payload.
Duplicate?
Not Duplicate

#### Cons
umer
Receive Message
Extract Message ID
Check Deduplication DB
Backpressure (DiscardMessage)
Process Message
Send ACK to Broker

#### Implementation

Use a message broker that supports built-in deduplication and backpressure features, such as Kafka with its acks and retries settings.
Implement custom logic using libraries like Nats or custom middleware in your application.

#### Closing Summary

By implementing a robust backpressure mechanism and deduplication strategy, we can effectively mitigate the risk of retry-amplification during broker upgrades. This ensures that downstream services do not receive excessive requests due to retries, preventing storms and system failures.

#### Common Mistakes/Follow-Ups

Not Considering Capacity: Failing to account for the varying processing capacities of different consumers.
Ignoring Global State: Not considering a centralized state management for deduplication across multiple instances.
Overcomplicating Deduplication: Using overly complex deduplication strategies that add unnecessary overhead.

**Code:**

```python
# Example Python code to implement sliding window backpressure
class SlidingWindow:
    def __init__(self, max_messages=5, time_window=30):
        self.max_messages = max_messages
        self.time_window = time_window
        self.message_counter = 0

    def process_message(self):
        if self.message_counter < self.max_messages:
            self.message_counter += 1
            return True
        else:
            # Apply backpressure
            print("Backpressure applied due to sliding window limit.")
            return False
```

**Example input / output:**

```text
Input:
- Message received by consumer with ID "unique-id"

Output:
- Consumer checks message ID in database and finds it has already been processed.
- Consumer sends backpressure signal to broker and does not process the message again.
```

**Data:**

```json
{
  "message": {
    "id": "unique-id",
    "payload": "some-data"
  }
}
```

