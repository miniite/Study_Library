---
type: "practice"
difficulty: "Deep"
---

# Design backpressure for a message-driven service where the consumer is slower than the producer — compare bounded queues pausing consumption and load-shedding and how you signal producers to slow down.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Fan-out Trade-offs](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Fan-out_Trade-offs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design an effective backpressure mechanism for a message-driven service where the consumer lags behind the producer, we need to ensure that the system remains stable and responsive without overwhelming the consumer with too many messages at once. Two primary approaches can be considered here: bounded queues with pausing consumption and load-shedding. Each approach has its trade-offs and implications.

Triaging the Problem

The question is essentially asking us to design a backpressure mechanism that allows consumers to handle incoming messages more gracefully when they fall behind producers. We need to ensure that neither the consumer nor any part of the system gets overloaded, which could lead to crashes or performance degradation.

Key Assumptions:
Message Rate: The rate at which messages are produced is significantly higher than the consumption rate.

#### Cons
umer Capabilities: Consumers can process messages asynchronously and have some form of state management.
System Stability: Ensuring that the system remains stable under heavy load without dropping critical messages.

#### First-Principles Solution
Bounded Queues with Pausing Consumption
Queuing Mechanism:
Introduce a bounded queue between the producer and consumer. This ensures that only a limited number of messages can be stored in transit at any given time.
Monitoring Queue Length:
Continuously monitor the length of this queue. When it exceeds a certain threshold, indicate to the producer to pause or throttle message production.
Producer Feedback Loop:
Implement a feedback loop where producers receive signals from consumers about the queue length and adjust their rate of message generation accordingly.
Load-Shedding
Selective Message Dropping:
Instead of pausing consumption, selectively drop messages when the consumer cannot keep up. This approach ensures that critical or less important messages can still be processed.
Prioritization and Retention Policies:
Prioritize message types based on their importance and design retention policies to ensure that only non-critical messages are dropped.
Detailed Implementation
Bounded Queues with Pausing Consumption
Queue Setup:
PYTHON
from queue import Queue
bounded_queue = Queue(maxsize=100)
Producer Code:
PYTHON
while True:
    if bounded_queue.full():
        # Producer pauses or throttles message production
        producer_pause()
    else:
        # Add new messages to the queue
        message = generate_message()
        bounded_queue.put(message)

#### Cons
umer Code:
PYTHON
while True:
    if not bounded_queue.empty():
        message = bounded_queue.get()
        process_message(message)
    else:

#### cons
umer_sleep()  # Consumer waits until more messages arrive
Load-Shedding
Selective Dropping:
PYTHON
def drop_less_important_messages():
    while True:
        if not bounded_queue.empty():
            message = bounded_queue.get()
            if is_important_message(message):
                process_message(message)
            else:
                # Drop less important messages
                discard_message(message)
        else:

#### cons
umer_sleep()  # Consumer waits until more messages arrive

#### Closing Summary
and Common Mistakes/Follow-Ups

#### Common Mistakes

Failing to implement a proper feedback mechanism that signals producers when the queue is full.
Overlooking the importance of prioritizing messages correctly, which can lead to critical data being dropped.

#### Follow-Up
Questions:

How do you ensure fairness in load-shedding scenarios?
What are the implications on system latency and throughput?
How would you integrate this mechanism with other components like retries or dead-letter queues?

By carefully considering these trade-offs, we can design a robust backpressure mechanism that ensures both consumer and producer stability in a message-driven service.

**Comparison:**

```
Mechanism	Queue Management	Consumer Feedback
Bounded Queues	Pauses or throttles producers.	Uses queue length to signal producers.
Load-Shedding	Selectively drops messages.	Prioritizes and retains critical messages.
```

**Code:**

```python
# Bounded Queue Example
from queue import Queue

bounded_queue = Queue(maxsize=100)

def producer_pause():
    # Logic to pause or throttle producer
    pass

def generate_message():
    return "Sample Message"

def process_message(message):
    print(f"Processing: {message}")

def consumer_sleep():
    time.sleep(0.5)

while True:
    if bounded_queue.full():
        producer_pause()
    else:
        message = generate_message()
        bounded_queue.put(message)
```

**Example input / output:**

```text
Input:
- Producer sends messages at a rate faster than consumer processing.

Output:
- Bounded queue fills up, producer pauses.
- Consumer processes messages until queue empties and then resumes consumption after pausing.
```

**Data:**

```json
{
  "queue": {
    "maxsize": 100,
    "messages": ["Message1", "Message2", "Message3"]
  }
}
```

**Flow:**

1. Yes
2. No
3. No
4. Yes
5. Producer
6. Queue Full?
7. Pause/Throttle
8. Generate Message
9. Add to Queue
10. Consumer
11. Queue Empty?
12. Process Message
13. Sleep/wait for more messages

