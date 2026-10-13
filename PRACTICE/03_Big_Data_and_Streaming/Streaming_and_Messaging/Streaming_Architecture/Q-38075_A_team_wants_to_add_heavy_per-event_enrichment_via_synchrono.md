---
type: "practice"
difficulty: "Deep"
---

# A team wants to add heavy per-event enrichment via synchronous calls to a downstream service inside a streaming operator. Explain why that's an anti-pattern at scale and design the async enrichment architecture you'd mandate instead.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Log-first vs Processor-first Platforms](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Provisional_and_Final_Results/Log-first_vs_Processor-first_Platforms.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary

When implementing real-time streaming architectures, performing heavy per-event enrichment synchronously within a streaming operator can lead to significant bottlenecks and scalability issues. By designing an asynchronous enrichment architecture, we ensure that each event is processed independently without blocking the main stream, allowing for better performance and maintainability.

#### Triaging the Question

The core of this question revolves around understanding why synchronous calls inside a streaming operator are problematic at scale and then designing an alternative solution to handle per-event enrichment asynchronously. The interviewer is probing your knowledge of streaming architectures, concurrency handling, and trade-offs between synchronous and asynchronous processing.

#### Assumptions
and Reasoning from First Principles
Synchronous Calls: When you make a synchronous call within the streaming operator, it stops further processing until the response from the downstream service is received.
Concurrency Issues: Synchronous calls can lead to thread blocking in a multi-threaded environment, causing delays and potential deadlocks.
Scalability: As more events are processed, the number of synchronous calls increases, leading to exponential growth in latency and resource usage.

#### First-Principles Solution

To address these issues, we need an asynchronous enrichment architecture that allows each event to be independently enriched without blocking other events. This can be achieved using a message queue system or a task-based processing model where enrichment tasks are queued up for later execution.

Step 1: Identify the Enrichment Service

Determine what kind of enrichment is needed and how it will interact with the streaming pipeline. For instance, if the service needs to perform complex logic like querying databases or performing API calls, these operations should be offloaded from the main stream.

Step 2: Design a Message Queue System

Introduce a message queue (like RabbitMQ, Kafka, etc.) between the streaming operator and the enrichment service. This allows events to be dispatched to the queue without blocking the main pipeline.

Stream
Message Queue
Enrichment Service
Step 3: Implement Event-Driven Processing

The enrichment service should process messages from the queue independently. This could be done using a worker pool or a microservice architecture where each instance can handle multiple events in parallel.

Stream
Message Queue
Worker Pool/Service
Step 4: Handle Backpressure and Throttling

Ensure that the enrichment service has mechanisms to manage backpressure. For example, rate limiting can prevent overwhelming the queue with too many requests from a single stream.

#### Closing Summary
and Common Mistakes/Follow-Ups

By implementing an asynchronous architecture, we avoid blocking the main streaming operator, ensuring better performance and scalability. However, common mistakes include underestimating the complexity of managing message queues and not adequately handling backpressure scenarios.

#### Follow-Up
Questions
How would you implement retry mechanisms for failed enrichments?
What are some strategies to optimize the throughput of your enrichment service?

**Code:**

```python
# Example Python code snippet using a simple async queue
import asyncio

async def enrich_event(event):
    # Simulate heavy computation or API call
    await asyncio.sleep(1)
    return f"Enriched {event}"

async def worker(queue):
    while True:
        event = await queue.get()
        if event is None:
            break
        enriched_event = await enrich_event(event)
        print(enriched_event)
        queue.task_done()

# Example stream processing with async calls
stream_events = ["Event1", "Event2", "Event3"]

async def main():
    queue = asyncio.Queue()

    # Start worker threads
    for _ in range(5):
        asyncio.create_task(worker(queue))

    # Enrich events asynchronously
    tasks = [queue.put(event) for event in stream_events]
    await asyncio.gather(*tasks)

    # Signal workers to exit
    for _ in range(5):
        queue.put_nowait(None)
    await queue.join()

# Run the main function
asyncio.run(main())
```

**Example input / output:**

```plaintext
Input:
stream_events = ["Event1", "Event2", "Event3"]

Output:
[
  "Enriched Event1",
  "Enriched Event2",
  "Enriched Event3"
]
```

**Data:**

```json
{
  "stream_events": ["Event1", "Event2", "Event3"],
  "enriched_output": [
    "Enriched Event1",
    "Enriched Event2",
    "Enriched Event3"
  ]
}
```

**Flow:**

1. Stream
2. Message Queue
3. Enrichment Service
4. Worker Pool/Service
5. ###

