---
type: "practice"
difficulty: "Intermediate"
---

# What are the pros and cons of event-driven architecture compared to a request-response monolith?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Breaking Schema Changes](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Fat_vs_Thin_Events/Breaking_Schema_Changes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Event-driven architectures (EDA) and request-response monolithic systems each have their own strengths and weaknesses, particularly when it comes to scalability, fault tolerance, and responsiveness. In this response, I'll outline the pros and cons of EDA versus a request-response monolith from an intermediate engineer's perspective.

TL;DR

#### Pros
of Event-Driven Architecture (EDA): Scalability, loose coupling, better fault isolation, improved performance for IO-bound tasks.

#### Cons
of EDA: Complexity in design and implementation, potential latency issues due to asynchronous processing, increased memory usage from message queuing.

#### Triage
Understanding the Question

The question is asking you to compare an event-driven architecture with a request-response monolith. The interviewer might be probing your understanding of the architectural trade-offs involved in choosing one approach over another. You need to provide both high-level and detailed insights, covering aspects like system responsiveness, fault tolerance, complexity, performance, and ease of development.

#### First-Principles Solution

Let's start by defining the two architectures:

Request-Response Monolith:

A monolithic application where a single thread or process handles requests sequentially.
Requests are synchronous; the client waits for the response before continuing.
The system is tightly coupled, making changes and scaling difficult.

Event-Driven Architecture (EDA):

An architecture where components communicate by sending events to each other over message queues or topics.
Events can be processed asynchronously, allowing systems to scale horizontally more easily.
Components are loosely coupled, enabling better fault isolation and independent scaling of different services.

#### Pros
of Event-Driven Architecture
Scalability:
EDA allows you to scale individual components independently since they do not wait for each other.
Loose Coupling:
Services can evolve without impacting others, making the system more resilient and easier to maintain.
Fault Isolation:
Failures in one component do not necessarily bring down the entire system.
Improved Performance:
IO-bound tasks (like reading from a database or API calls) are better offloaded to background workers, improving overall throughput.

#### Cons
of Event-Driven Architecture
Complexity:
Designing and implementing an EDA can be more complex due to the additional layers of messaging infrastructure.
Latency Issues:
Asynchronous processing means that responses might take longer to arrive, especially if there are multiple hops.
Increased Memory Usage:
Message queues and topics consume memory, which can become a bottleneck in large-scale systems.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, EDA offers significant benefits like scalability, loose coupling, and improved fault tolerance but comes with added complexity and potential performance overhead. A request-response monolith is simpler to design and implement but may struggle to scale and handle complex, multi-service interactions efficiently.

#### Common mistakes
include underestimating the complexity of implementing EDA or overestimating the ease of migrating from a monolithic architecture. It's crucial to understand the specific requirements of your application before deciding which approach to use.

For follow-ups, you might be asked about specific tools (like RabbitMQ or Kafka) used in EDA, the trade-offs between synchronous and asynchronous communication patterns, or how to mitigate latency issues in an EDA.

**Comparison:**

```
Type	Request-Response Monolith	Event-Driven Architecture (EDA)
Scalability	Limited; hard to scale out	Easy; components can be scaled independently
Coupling	Tight, changes impact others	Loose, services are decoupled
Fault Tolerance	Poor; one failure can bring down the system	Good; failures isolated to individual components
Performance	Better for CPU-bound tasks	Better for IO-bound tasks, background processing
```

**Code:**

```python
# Example of a simple producer-consumer setup using Python's asyncio for an event-driven system

import asyncio
from random import randint

async def produce(queue):
    """Produce items asynchronously"""
    while True:
        item = randint(1, 10)
        await queue.put(item)
        print(f"Produced: {item}")
        await asyncio.sleep(randint(1, 3))

async def consume(queue):
    """Consume items asynchronously"""
    while True:
        item = await queue.get()
        print(f"Consumed: {item}")
        queue.task_done()

# Example setup
queue = asyncio.Queue()
producer_task = asyncio.create_task(produce(queue))
consumer_task = asyncio.create_task(consume(queue))

asyncio.run(asyncio.gather(producer_task, consumer_task))
```

**Example input / output:**

```text
Input:
- User submits an order (HTTP request to API endpoint)

Output:
- Order creation event sent to message queue -> Order processing service picks up the event and starts a background task to process the order.

Dry Run:
1. User submits an order.
2. The order creation event is published to the "orders" topic.
3. An order processing worker reads the event from the queue, processes it in the background, and updates the database.
4. Order status changes to "processing".
```

**Data:**

```json
{
  "messageQueue": {
    "topic": "orders",
    "messages": [
      { "id": 1, "type": "order_created", "data": {"userId": 101, "productId": 202} },
      { "id": 2, "type": "order_updated", "data": {"userId": 101, "status": "processing"} }
    ]
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. User submits an order
6. Is monolith?
7. Request processedsynchronously
8. Subscribe to topic 'orders'?
9. Process event in background
10. Send event to queue

