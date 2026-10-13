---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd cap the tail latency (p99) of an async pipeline where most messages are fast but 1% call a slow external API. How do you keep the fast ones from queueing behind slow ones?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Lag-based Consumer Autoscaling](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Lag-based_Consumer_Autoscaling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To tackle the challenge of capping tail latency (p99) in an async pipeline where most messages are processed quickly but 1% call a slow external API, we need to ensure that fast messages don't get queued behind these slow ones. This involves designing a system with multiple layers of queuing and processing strategies.

#### Triaging the Question

The question is essentially asking how to design a highly available and performant async pipeline where the overall tail latency can be managed effectively. The interviewer is probing your understanding of:

Message queues and their various implementations.
How to handle different message latencies in an event-driven architecture.
Scalability, fault tolerance, and performance trade-offs.
Techniques to manage backpressure and ensure that fast messages are not delayed by slow ones.

#### Assumptions
We have a system where most messages (99%) process quickly.
A small fraction of messages (1%) may take significantly longer due to an external API call.
The goal is to minimize the tail latency while ensuring high throughput and low response times for all messages.

#### Solution
from First Principles
Step 1: Designing Queues

We can use a combination of different queues to segregate fast and slow messages effectively:

Fast Queue: This queue will handle the majority of messages that are processed quickly.
Slow Queue: This queue will specifically handle the slower messages, ensuring they don’t delay faster ones.
Step 2: Implementing Backpressure Management

To manage backpressure, we need to ensure that fast queues do not get blocked by slow ones:

Use a sliding window mechanism to monitor the latency of each queue.
If the slow queue starts queuing up, introduce additional processing threads or parallelism for it.
Step 3: Prioritization and Throttling

Prioritize messages based on their urgency:

Urgent Messages: Process these first using dedicated threads.
Regular Messages: Handle these in a balanced manner without overwhelming the system.
Step-by-Step Solution

Queue Setup:

Use a message broker like RabbitMQ or Apache Kafka to set up two queues: one for fast messages and another for slow ones.
Process Quickly
Handle Slowly
FastQueue
Worker1
SlowQueue
Worker2

Backpressure Handling:

Monitor the latency of both queues using a metric system like Prometheus or Grafana.
If the slow queue starts to back up, dynamically increase its processing capacity by adding more workers or scaling out.

Prioritization Strategy:

Use a priority-based message routing mechanism where urgent messages are prioritized and processed first.
High Priority
Medium Priority
UrgentMessages
FastQueue
RegularMessages
SlowQueue

Monitoring and Logging:

Continuously monitor the latency of each queue and log any anomalies for troubleshooting.
Use tools like ELK Stack (Elasticsearch, Logstash, Kibana) to visualize and analyze message latencies.

#### Closing Summary

By using a combination of segregated queues, backpressure management, prioritization strategies, and dynamic scaling, we can effectively manage the tail latency in an async pipeline. This approach ensures that fast messages are not delayed by slower ones, thereby improving overall system performance and responsiveness.

#### Common Mistakes
and Follow-Ups

Mistake 1: Failing to implement proper backpressure management leading to a backlog of slow messages.

#### Follow-Up
Discuss how different queuing strategies can impact the overall throughput and latency. For example, using a bounded buffer queue or a priority-based scheduler.

Mistake 2: Not prioritizing urgent messages adequately, which could lead to significant delays in critical operations.

#### Follow-Up
Explore alternative scheduling algorithms like Round Robin with priorities or Fair Queuing to ensure balanced processing of all types of messages.

**Comparison:**

```
Queue Type	Message Processing Time (ms)	Expected Latency (p99)
FastQueue	<5	~2
SlowQueue	300	~300
```

**Code:**

```python
# Example Python code for dynamic scaling using a simple threshold-based approach
import threading

def monitor_queue(queue, threshold=10):
    while True:
        latency = queue.get_latency()
        if latency > threshold:
            add_workers(queue)

def add_workers(queue):
    # Logic to scale out the number of workers processing slow messages
    pass

# Example setup for fast and slow queues
fast_queue = ...  # Setup FastQueue with high throughput
slow_queue = ...  # Setup SlowQueue with lower latency monitoring
threading.Thread(target=monitor_queue, args=(slow_queue,)).start()
```

**Example input / output:**

```text
Input (Example):
100 messages -> FastQueue, SlowQueue

Output:
99% of messages processed quickly from FastQueue.
1% of messages delayed in SlowQueue but do not affect the tail latency.

Dry-run:
- FastQueue processes 99 messages within 5ms each.
- SlowQueue processes 1 message taking 300ms due to external API call.
```

**Data:**

```json
{
  "queue": {
    "type": "RabbitMQ",
    "configuration": {
      "host": "localhost",
      "port": 5672,
      "vhost": "/",
      "username": "guest",
      "password": "guest"
    },
    "queues": [
      { "name": "fastQueue", "type": "direct" },
      { "name": "slowQueue", "type": "direct" }
    ]
  }
}
```

**Flow:**

1. No
2. Yes
3. Backpressure - F[Add MoreWorkers];D
4. Message Arrival
5. Is Message Urgent?
6. Process in FastQueue
7. Process in SlowQueue
8. Monitor Latency
9. Monitor Latency

