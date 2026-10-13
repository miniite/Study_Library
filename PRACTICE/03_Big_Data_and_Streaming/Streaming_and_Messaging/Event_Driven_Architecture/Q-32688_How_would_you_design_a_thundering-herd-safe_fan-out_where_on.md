---
type: "practice"
difficulty: "Deep"
---

# How would you design a thundering-herd-safe fan-out where one event triggers work for a million users?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Lag-based Consumer Autoscaling](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Lag-based_Consumer_Autoscaling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a thundering-herd-safe fan-out system where an event triggers work for a million users, we need to ensure that each user receives their message only once and in order, while scaling efficiently. This involves understanding the key components of the architecture—message queues, event-driven systems, and distributed processing. We'll break this down from first principles.

Firstly, let's clarify what "thundering herd" means: it refers to a scenario where many users are triggered simultaneously, potentially overwhelming a system with too many concurrent requests. In our case, we need to ensure that each user gets their message exactly once and in the correct order, even when thousands of events trigger work for millions of users.

We'll start by breaking down the problem into manageable components:

Event Source: The source of the event (e.g., a server or microservice) needs to be able to handle high concurrency.
Message Queueing System: A message queue like RabbitMQ, Kafka, or Amazon SQS will handle fan-out and ensure that each message is processed exactly once.
Worker Services: These services will consume messages from the queue and process them.

Let's dive into each component in more detail:

Event Source:

The event source should generate events asynchronously to avoid blocking.
It can use a database transaction or distributed lock mechanism to ensure that an event is not processed multiple times if there are concurrent requests.

Message Queueing System:

We'll choose a reliable message queue system that supports partitioning and fault tolerance, like Kafka for its high throughput and durability.
Messages will be sent to a topic where each user has a dedicated consumer group or partition.
The at-least-once delivery semantics of the queue ensure that messages are not lost, but duplicates might occur. We can use sequence numbers in the message payload to handle this.

Worker Services:

Each worker service will subscribe to a specific user's message queue and process messages one by one.
To avoid thundering herd, we'll implement a backpressure mechanism where workers dynamically adjust their processing rate based on the number of pending messages.
A load balancer can distribute incoming events evenly across multiple instances of these services.

Now let's reason step-by-step to design this system:

Event Source:

Implement the event source with non-blocking I/O to handle high concurrency.
Use a distributed lock (e.g., Redis) to ensure that an event is processed only once during concurrent events.

Message Queueing System:

Set up Kafka brokers and topics for each user.
Configure the producers to send messages to the appropriate topic for each user.
Ensure at-least-once delivery semantics using Kafka's partitioning and acknowledgments.
Use sequence numbers in the message payload to handle duplicate processing.

Worker Services:

Deploy worker services on multiple machines or containers, managed by a service mesh like Istio for load balancing and resilience.
Implement backpressure logic: if the queue backlog exceeds a threshold, reduce the processing rate; otherwise, increase it.
Use a distributed state store (e.g., Consul) to maintain global state and configuration.

#### Closing Summary
In summary, designing a thundering-herd-safe fan-out system involves leveraging reliable message queues like Kafka, ensuring that each user receives messages exactly once, and implementing backpressure mechanisms in worker services. This approach ensures scalability and fault tolerance while handling high concurrency efficiently.

#### Common Mistakes/Follow-Ups

Duplicate Processing: Ensure that messages are processed only once using sequence numbers.
Backpressure Logic: Implementing dynamic load adjustment based on queue length to avoid overwhelming the system.
Fault Tolerance: Use distributed systems like Kafka and service meshes for resilience.

**Comparison:**

```
Worker	Queue Length	Processing Rate
W1	10	2 messages/s
W2	20	4 messages/s
W3	5	1 message/s
```

**Code:**

```
Example producer code to send a message to a Kafka topic

from kafka import KafkaProducer

producer = KafkaProducer(bootstrap_servers='localhost:9092') message = b'{"user_id": 12345, "event_data": "data"}' future = producer.send('user_12345_topic', key=b'user_key', value=message)

CODE
Copy
```python
# Example consumer code to process messages in a worker service
from kafka import KafkaConsumer

consumer = KafkaConsumer('user_12345_topic', group_id='worker_group')
for message in consumer:
    print(f"Processing: {message.value}")
    # Process the message using backpressure logic
```

**Example input / output:**

```

Input:

An event is generated with user data. Output:
Each relevant worker service receives and processes the message exactly once.

Request:

Send a message to the Kafka topic for user 12345. Response:
JSON
Copy
{
  "event": {
    "userId": 12345,
    "data": "user update"
  }
}
```

**Data:**

```json
{
  "event": {
    "userId": 12345,
    "data": "user update"
  }
}
```

**Flow:**

1. Message
2. Backpressure
3. Backpressure
4. Event Source
5. Generate Event
6. Message Queue (Kafka)
7. Worker Service 1
8. Worker Service 2
9. Process Message
10. Process Message
11. Adjust Processing Rate
12. Adjust Processing Rate

