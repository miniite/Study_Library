---
type: "practice"
difficulty: "Deep"
---

# A consumer must fan out one incoming event into per-recipient work for up to 5 million recipients. How do you avoid a single giant transaction and keep the fan-out restartable?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Idempotent Consumers](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Fat_vs_Thin_Events/Idempotent_Consumers.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing an architecture where a single incoming event needs to be fanned out into per-recipient work for up to 5 million recipients, the primary goal is to avoid creating a single giant transaction that could lead to data inconsistencies or failures. This problem requires careful consideration of state management, concurrency, and fault tolerance—especially given the scale involved. To achieve this, we need to employ message queues and event-driven architecture principles.

First, let's break down what the interviewer might be probing: understanding of distributed systems, awareness of transactional integrity in large-scale applications, familiarity with state management strategies, and knowledge of how to handle failures gracefully. We'll assume that each recipient is uniquely identified by a distinct key, such as an email address or user ID.

To solve this problem, we can employ a combination of message queues (e.g., RabbitMQ, Apache Kafka) and event-driven architecture principles. The core idea is to split the fan-out process into multiple smaller transactions, ensuring that each recipient gets its own task. We'll use a distributed state management strategy to track the processing status and ensure restartability.

Message Queue Setup: Use a message queue where the incoming event triggers the production of messages for each recipient. Each message in the queue will contain both the event payload and the unique identifier of the recipient.
Worker Nodes: Deploy multiple worker nodes that consume messages from the queue, process them independently, and send results to their respective recipients. This decouples the processing logic from the event source and ensures scalability.
State Management with Kafka Streams/KSQL or Distributed Stateful Processing in Apache Flink: Utilize a stream-processing framework to maintain state across worker nodes. For example, you can use Kafka Streams to process each incoming message and update the state (e.g., mark the recipient as processed) atomically. Alternatively, Apache Flink provides robust support for distributed stateful processing.

Here is a step-by-step reasoning from first principles:

Initial Setup:
The application receives an event that needs to be fanned out.
Each worker node (e.g., microservices) connects to the message queue and starts listening for incoming messages.
Message Production:
When a new event arrives, it is transformed into multiple messages, each with a unique recipient identifier.
These messages are published to the queue in parallel or serially based on system load and resource availability.
Worker Node Consumption:
Each worker node consumes messages from the queue.
The message contains both the event payload and the recipient identifier.
State Management:
As each message is processed, state about its processing status (e.g., whether it has been sent to the recipient) needs to be updated.
Use a distributed state management system like Kafka Streams or Apache Flink to ensure that this state is consistent across all worker nodes and resilient in case of failures.
Error Handling & Restartability:
Implement idempotent processing: if a message is processed multiple times, it should have no side effects beyond the initial processing.
Ensure that any failure can be retried without causing data inconsistencies or losing messages.

By employing these strategies, we ensure that the system remains scalable and resilient. The use of stateful stream-processing frameworks ensures that even in case of failures, the processing status is maintained correctly, allowing for a restartable architecture.

**Comparison:**

| Recipient ID | Event Payload |
|--------------|---------------|
| user123      | some event data  |

**Code:**

```python
# Example code using Apache Flink for event processing

from pyflink.dataset import ExecutionEnvironment
from pyflink.table import StreamTableEnvironment, DataTypes
from pyflink.table.descriptors import Schema, OldCsv, FileSystem

env = ExecutionEnvironment.get_execution_environment()
t_env = StreamTableEnvironment.create(env)

# Define the schema of input and output files
t_env.connect(FileSystem().path('input'))
     .with_format(OldCsv()
                  .field('recipient_id', DataTypes.STRING())
                  .field('event_payload', DataTypes.STRING()))
     .with_schema(Schema()
                  .field('recipient_id', DataTypes.STRING())
                  .field('event_payload', DataTypes.STRING()))
     .register_input('input')

t_env.connect(FileSystem().path('output'))
     .with_format(OldCsv()
                  .field('status', DataTypes.STRING()))
     .with_schema(Schema()
                  .field('status', DataTypes.STRING()))
     .register_output('output')

# Define the processing logic
t_env.from_path('input') \
     .group_by(...) \
     .select(...)
```

**Example input / output:**

```text
Input:
- Event received with payload and recipient list.

Output:
- Multiple messages produced for each recipient.
- Worker nodes process each message independently.
```

**Data:**

```json
{
  "recipient_id": "user123",
  "event_payload": "some event data"
}
```

**Flow:**

1. Yes
2. No
3. Event Received
4. Transform to MultipleMessages
5. Queue Full?
6. Wait for Queue Space
7. Send Messages to Queue
8. Worker Nodes ConsumeMessages
9. Process Event and UpdateState

