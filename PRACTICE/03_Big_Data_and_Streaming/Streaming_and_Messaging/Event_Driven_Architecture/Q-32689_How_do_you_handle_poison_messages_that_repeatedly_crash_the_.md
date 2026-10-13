---
type: "practice"
difficulty: "Deep"
---

# How do you handle poison messages that repeatedly crash the consumer process itself, not just fail logically?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Poison Messages and DLQs](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Poison_Messages_and_DLQs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To handle poison messages in an event-driven architecture with message queues, we need to ensure robustness and resilience in our system design. Poison messages are problematic because they can repeatedly crash the consumer process, leading to downtime or unstable operations. The goal is to isolate these problematic messages and prevent them from bringing down the entire system while ensuring that legitimate business logic can continue.

#### Triage
Understanding the Question

The core of this question revolves around designing a mechanism to manage poison messages without crashing the consumer process. The interviewer might be probing for your understanding of error handling, message reprocessing strategies, and the impact on system reliability and performance. Assumptions we can make include:

You are familiar with common message queue systems like RabbitMQ or Kafka.
You understand basic event-driven architectures.
You recognize that poison messages could stem from data corruption, unexpected formats, or logic errors.

#### First Principles Solution
Message Validation: Implement robust validation checks at the consumer level to detect and reject invalid messages.
Isolation of Failed Consumers: Use a separate error handling mechanism to manage failed consumers without affecting other processes.
Retry Mechanism: Introduce a retry strategy for failing messages, but with a limited number of attempts before marking them as poison.
Dead Letter Queue (DLQ): Send poison messages to a DLQ where they can be inspected and handled separately.
Step-by-Step Solution

Message Validation:

Implement a pre-processing step in the consumer that validates message fields against predefined schemas or expected formats.

#### Example
If you expect a JSON payload, validate its structure using JSON schema validation tools before processing.

Error Handling with Dead Letter Queues (DLQ):

Set up a DLQ where messages that fail multiple retries are sent for further inspection and manual handling.
This prevents the consumer from crashing and allows for manual intervention to resolve issues like data corruption or logic flaws in the producer.

Retry Mechanism:

Configure message queues to allow retries with exponential backoff or jitter to prevent overwhelming the system.

#### Example
A typical retry policy might involve three attempts, each with a delay of 10 seconds, then 2 minutes, and finally 4 minutes before sending the message to the DLQ.

Logging and Monitoring:

Implement comprehensive logging for failed messages in both the consumer and queue systems.
Use monitoring tools to track retry counts and other metrics that can help identify patterns or issues with specific messages or producers.

Manual Intervention:

Establish a process for reviewing and addressing poison messages in the DLQ.
This could involve setting up alerts, creating a ticketing system, or integrating with another tool like Slack or Jira to notify relevant teams.

#### Closing Summary

By implementing message validation, using dead letter queues, configuring retry mechanisms, and setting up robust logging and monitoring, we can effectively handle poison messages without crashing the consumer process. This ensures that the main workflow continues unimpeded while allowing for manual intervention when necessary.

#### Common Mistakes
and Follow-Ups
Misinterpreting Retry Logic: Failing to implement proper retry logic can lead to flooding of a DLQ with messages, overwhelming the system.
Overlooking Validation: Simple validation steps might be overlooked, leading to undetected issues that manifest as poison messages later on.
No Manual Process for DLQ: Ignoring the need for manual intervention in the DLQ can result in unresolved issues and potential data loss.

For a deeper dive, common follow-ups could involve discussing the trade-offs between different retry policies, the impact of scaling these mechanisms, and best practices for managing large volumes of messages.

**Comparison:**

```
Retry Attempt	Delay (s)
1	10
2	60
3	120
```

**Code:**

```python
# Example Python code snippet for message validation using jsonschema

import jsonschema
from json import loads

def validate_message(message):
    schema = {
        "type": "object",
        "properties": {
            "id": {"type": "integer"},
            "data": {"type": "string"}
        },
        "required": ["id", "data"]
    }

    try:
        message_json = loads(message)
        jsonschema.validate(message_json, schema)
        return True
    except jsonschema.exceptions.ValidationError as err:
        print(f"Validation error: {err}")
        return False

# Example usage in a consumer
if not validate_message(received_message):
    # Handle the invalid message (e.g., log it or send to DLQ)
    pass
```

**Example input / output:**

```text
Input: {"id": "notAnInteger", "data": "exampleData"}

Output:
Validation error: 'id' should be integer
```

**Data:**

```json
{
  "validMessage": {
    "id": 1,
    "data": "exampleData"
  },
  "invalidMessage": {
    "id": "notAnInteger",
    "data": "exampleData"
  }
}
```

**Flow:**

1. Yes
2. No
3. No
4. Yes
5. Receive Message
6. Is Valid?
7. Process Message
8. Log Error
9. Retry?
10. Send to DLQ

