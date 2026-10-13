---
type: "practice"
difficulty: "Expert"
---

# Design a canary/rollout strategy for changing a topic's delivery semantics (at-least-once to exactly-once) in production without a flag day, given hundreds of independently-deployed consumers with mixed readiness.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Dedup Contracts](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Delivery_Semantics/Dedup_Contracts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When redesigning message delivery semantics from at-least-once to exactly-once in a distributed system, ensuring minimal disruption and gradual transition is crucial. This can be achieved through a sophisticated canary rollout strategy that allows us to test the new semantics without affecting all consumers immediately. Here’s how we would approach this problem:

#### Triage
What's Being Probed

The interviewer is probing our ability to design a robust system upgrade process, considering factors like consumer readiness, data consistency, and operational complexity. We need to ensure that no critical failures occur during the transition, and that we can rollback if necessary.

#### Assumptions

#### Cons
umer Readiness: Some consumers are more ready than others for the new semantics.
Topic Data Volume: There is a significant amount of data being processed through the topic.

#### Cons
umer Heterogeneity: Consumers are independently deployed, some may be slow to update, and some might not even support the exact-once semantics.

#### First Principles Solution
Versioned Semantics: Implement versioned message handling logic in both old and new consumers. This allows us to send messages with a version tag that dictates whether it should adhere to at-least-once or exactly-once guarantees.
Canary Consumers: Start by deploying the updated semantics on a subset of consumers. These canaries will help identify any issues without impacting all consumers.
Monitoring and Feedback Loops: Continuously monitor system performance and gather feedback from both producers and consumers to ensure smooth operation.
Rollback Plan: Have a clear rollback plan in case the new semantics cause issues, ensuring minimal downtime.
Step-by-Step Solution

Versioned Messages:

Modify the producer to add a version field to each message, indicating its delivery semantics (e.g., "at-least-once" or "exactly-once").
PYTHON
class Message {
    version: str  # either 'at-least-once' or 'exactly-once'
    payload: Any   # actual data payload
}

#### Cons
umer Logic:

#### Cons
umers should be updated to handle both versions of messages.
New consumers will process "exactly-once" messages differently, ensuring idempotency and atomicity.
PYTHON
def consume(message: Message):
    if message.version == 'at-least-once':
        # Handle as before
        ...
    elif message.version == 'exactly-once':
        handle_exactly_once(message)

def handle_exactly_once(message: Message):
    # Ensure idempotency and atomicity, e.g., using transactions or locks
    ...

Canary Deployment:

Gradually deploy the new consumers to a subset of the existing consumer pool.
PLAINTEXT
Step 1: Deploy canary consumers (10%)
Step 2: Monitor system for stability and gather feedback
Step 3: Roll out additional canaries as needed

Monitoring:

Implement comprehensive monitoring to track message delivery success rates, error rates, and latency.
PLAINTEXT
Metrics to monitor:
- Message delivery success rate
- Error rate for different versions of messages
- Latency in processing new vs old semantics

Feedback Loop:

Collect feedback from consumers about their readiness for the new semantics.
Use this information to make adjustments as needed during the rollout.

Rollback Plan:

Develop a rollback plan that allows reverting all consumers back to at-least-once semantics if issues arise.
PLAINTEXT
Rollback steps:
- Stop deployment of canary consumers
- Update remaining consumers to fallback logic

#### Closing Summary
+ Common Mistakes/Follow-Ups

This approach ensures a smooth transition by leveraging versioned messages and gradual consumer updates. It minimizes risk by starting with a small subset of consumers, monitoring performance closely, and having a clear rollback plan.

#### Common Mistakes

Failing to include a robust monitoring system.
Not planning for sufficient feedback loops from deployed canaries.
Overlooking the importance of a well-defined rollback strategy.

#### Follow-ups
could involve deeper discussions on specific technologies used (e.g., Kafka, RabbitMQ), potential challenges with idempotency in exactly-once semantics, and strategies to handle mixed consumer readiness more effectively.

**Comparison:**

```
Metric	Description
Success Rate	Percentage of messages delivered
Error Rate	Number of errors per message sent
Latency	Time taken to process a message
```

**Code:**

```python
class Message {
    version: str  # either 'at-least-once' or 'exactly-once'
    payload: Any   # actual data payload
}

def consume(message: Message):
    if message.version == 'at-least-once':
        # Handle as before
        ...
    elif message.version == 'exactly-once':
        handle_exactly_once(message)

def handle_exactly_once(message: Message):
    # Ensure idempotency and atomicity, e.g., using transactions or locks
    ...
```

**Example input / output:**

```text
Input:
- Producer sends a message with version="exactly-once".

Output:
- Canaries (10% of consumers) handle the message using new logic.
- Other consumers process it as usual.

Dry-run:
- Deploy canary consumers and monitor their behavior.
- Gradually increase canary deployment based on observed stability.
```

**Data:**

```json
{
  "messages": [
    {"version": "at-least-once", "payload": "data1"},
    {"version": "exactly-once", "payload": "data2"}
  ]
}
```

**Flow:**

1. Message Producer
2. Versioned Message
3. Canary Consumers (10%)
4. Failure Detection
5. Rollback Plan Activation
6. Main Consumer Base (90%)
7. Monitoring and Feedback
8. Continuous Improvement

