---
type: "practice"
difficulty: "Intermediate"
---

# You want to move a high-throughput synchronous enrichment call out of the request path into a queue, but the client still needs the enriched result. How do you keep it responsive?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Event-driven Order Processing](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Event-driven_Order_Processing.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this challenge, we need to decouple the high-throughput synchronous enrichment calls from the request path while ensuring that the client receives the enriched data in a timely manner. The key is to implement an event-driven architecture (EDA) with message queues. Here’s how it works:

TL;DR
Understand the Problem: We need to handle high-throughput requests without blocking the request path and ensure clients receive enriched results.

#### Solution
Overview: Use a message queue for asynchronous processing while maintaining responsiveness via an event-driven approach.

#### Triaging the Question

The core of this question is about efficiently handling incoming requests by offloading heavy computation tasks such as synchronous enrichment into a background process. The client should not experience delays or blocking, and must still get real-time updates on the enriched data. This requires careful design to balance between responsiveness and performance.

#### Assumptions
We have high-throughput synchronous calls that need to be offloaded.
Clients expect quick responses with enriched data.
We want to minimize downtime and ensure the system is resilient.

#### First Principles Solution
Message Queue for Asynchronous Processing: Use a message queue like RabbitMQ, Apache Kafka, or AWS SQS to handle incoming requests asynchronously. This way, the main request path remains light and quick.
Event-Driven Architecture: Once an enrichment task is offloaded, use events to notify the client about the status of the processing. For example, you could create a notification mechanism (e.g., a database trigger) that updates the client once the enriched data is ready.
Step-by-Step Solution

Client Request Handling:

The client sends an initial request with necessary input data.
The API endpoint captures this request and publishes it to a message queue.

Message Queue & Worker Processing:

A worker service listens for messages on the queue.
Upon receiving a message, the worker performs the heavy enrichment task (e.g., fetching additional data from multiple sources).
Once the processing is complete, an event or notification is sent back to the client.

Real-Time Client Updates:

The client can poll for updates or set up a long-polling mechanism.
Alternatively, use webhooks or push notifications to inform the client when data is ready.

Database Updates and Caching:

Update the database with the enriched results if needed.
Cache frequently accessed data in Redis or another in-memory store for faster retrieval by clients.

#### Closing Summary

By implementing an event-driven architecture with message queues, we can handle high-throughput requests efficiently while ensuring that clients receive timely and accurate enrichment data. The key is to separate concerns between the request path and background processing, leveraging modern infrastructure tools like message brokers and notifications.

#### Common Mistakes
& Follow-Ups
Blocking Client Polling: Avoid making the client wait by implementing webhooks or long-polling mechanisms.
Overloading Queues: Ensure proper queue management to avoid overloading and maintain performance.
Latency in Notifications: Optimize event notification times to ensure minimal delays.
Scalability & Resilience: Design for horizontal scaling of workers and consider failover strategies.

**Comparison:**

```
Message	Action Taken
New Request	Publish to Queue
Process Complete	Notify Client
```

**Code:**

```python
# Example worker service using RabbitMQ
import pika

def process_request(ch, method, properties, body):
    data = json.loads(body)
    # Perform heavy enrichment task
    enriched_data = perform_enrichment(data)

    # Notify client via database update or webhooks
    notify_client(enriched_data)

connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
channel = connection.channel()
channel.queue_declare(queue='enrichment_queue')
channel.basic_consume(queue='enrichment_queue', on_message_callback=process_request)
print('Waiting for requests. To exit press CTRL+C')
channel.start_consuming()
```

**Example input / output:**

```text
Client Request:
POST /enrich
{
  "id": "12345",
  "input_data": ["data1", "data2"]
}

Queue Message (sent to RabbitMQ):
{"id": "12345", "input_data": ["data1", "data2"]}

Worker Service Response:
POST /notify
{
  "id": "12345",
  "enriched_result": [
    { "name": "Data1", "value": "EnrichedValue1" },
    { "name": "Data2", "value": "EnrichedValue2" }
  ]
}
```

**Data:**

```json
{
  "request": {
    "id": "12345",
    "input_data": ["data1", "data2"]
  },
  "enriched_result": [
    { "name": "Data1", "value": "EnrichedValue1" },
    { "name": "Data2", "value": "EnrichedValue2" }
  ]
}
```

**Flow:**

1. Client Request
2. Queue
3. Worker Service
4. Enriched Result
5. Database Update/Caching
6. Notify Client

