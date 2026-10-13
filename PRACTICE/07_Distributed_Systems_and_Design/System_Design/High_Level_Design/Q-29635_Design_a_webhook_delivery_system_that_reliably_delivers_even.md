---
type: "practice"
difficulty: "Deep"
---

# Design a webhook delivery system that reliably delivers events to third-party endpoints — how do you handle retries, ordering, backoff, and slow/dead consumers?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Checkout System Design](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Checkout_System_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a reliable webhook delivery system, we need to focus on several key aspects: handling retries, maintaining message ordering, implementing backoff strategies, and dealing with slow or unresponsive endpoints. At a high level, our goal is to ensure that every event is delivered exactly once, even in the face of network failures, timeouts, and processing delays.

Triaging the Problem

The core question here involves creating a robust delivery system for webhooks. The interviewer is probing your understanding of distributed systems, reliability engineering, and how to handle various failure modes. Key assumptions include:

Webhook endpoints can fail temporarily or permanently.
Events must be delivered in order.
Some consumers might take longer than others to process events.
Network conditions vary unpredictably.

#### First Principles Solution
Message Queue: Use a reliable message queue service (e.g., RabbitMQ, Kafka) to buffer incoming events before they are sent to endpoints. This decouples producers from consumers and allows us to manage the flow of messages more effectively.
Retry Mechanism: Implement an exponential backoff strategy for retries when an endpoint fails. This helps avoid overwhelming a failing consumer while ensuring eventual delivery.
Ordering Guarantees: Ensure that messages are processed in the order they were received, which can be achieved using message sequences or timestamps.
Dead Letter Queue (DLQ): Create a DLQ to handle consumers that fail repeatedly. Messages in this queue can later be reprocessed manually or via another mechanism.
Health Checks: Implement health checks on consumer endpoints to detect failures and trigger retries or failover mechanisms.
Detailed Steps

Message Queuing:

Use a message broker like RabbitMQ or Kafka to store incoming events temporarily.

#### Cons
umers can then pull from the queue in an orderly fashion, reducing load spikes.

Retry Mechanism:

Define retry limits and backoff strategies (e.g., 1s, 2s, 4s, 8s).
Retry on transient errors only; permanent failures should be logged and escalated.
Use a database or cache to track retry attempts and deduplicate messages.

Ordering:

Assign unique IDs to each event.

#### Cons
umers can poll the queue in order by sequence ID, ensuring they process events in the correct order.

Backoff Strategy:

Implement an exponential backoff mechanism where failed deliveries are retried with increasing delays.
Use a circuit breaker pattern for consumers that fail frequently; this will automatically throttle retries and eventually fall back to manual intervention.

Dead Letter Queue (DLQ):

Configure the message broker to route messages from the main queue to a DLQ if they exceed retry limits.
Set up monitoring on the DLQ to identify problematic endpoints or consumers that need attention.

#### Closing Summary

By leveraging a combination of messaging queues, retry mechanisms, backoff strategies, and health checks, we can create a robust webhook delivery system capable of handling failures gracefully while ensuring reliable event delivery. The key is to design a flexible architecture that can adapt to different failure scenarios without compromising on performance or reliability.

#### Common Mistakes/Follow-ups
Not Handling Dead Consumers: Failing to implement a DLQ can lead to lost events if consumers stop processing.
Ignoring Performance Metrics: Without monitoring, it’s easy to overlook bottlenecks and issues that could impact delivery.
Overcomplicating Retry Logic: Overly complex retry strategies without proper backoff mechanisms can lead to resource exhaustion.

**Comparison:**

```
Event ID	Type	Payload
1	user.created	{...}
2	user.updated	{...}
```

**Code:**

```python
# Example Python code for exponential backoff using a decorator

import time
from functools import wraps

def backoff(max_retries=5, initial_delay=1, factor=2):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            retries = 0
            while retries < max_retries:
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    print(f"Error: {e}, retrying after delay...")
                    time.sleep(initial_delay * (factor ** retries))
                    retries += 1
        return wrapper
    return decorator

@backoff()
def send_webhook(url, payload):
    # Send webhook request and handle response
    pass
```

**Example input / output:**

```text
Input:
- Event: {"id": 1, "data": {"type": "user.created", "payload": {...}}}

Output:
- Queued event with ID: 1 in RabbitMQ.
- Sent to consumer for processing.

Dead Letter Queue (DLQ) Output:
- After retries fail, the event is moved to DLQ with a log entry.
```

**Data:**

```json
{
  "events": [
    {"id": 1, "data": {"type": "user.created", "payload": {...}}},
    {"id": 2, "data": {"type": "user.updated", "payload": {...}}}
  ]
}
```

**Flow:**

1. No
2. Yes
3. Receive Webhook Event
4. Queue Event in RabbitMQ
5. Event Processed?
6. Retry with Backoff
7. Route to DLQ if Failed
8. Log Delivery Success

