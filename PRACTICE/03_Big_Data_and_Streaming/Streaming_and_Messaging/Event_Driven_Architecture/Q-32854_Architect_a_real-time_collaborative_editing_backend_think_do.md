---
type: "practice"
difficulty: "Expert"
---

# Architect a real-time collaborative editing backend (think docs/whiteboard) on an event/log substrate where thousands edit the same document. How do ordering, conflict resolution, and per-document partitioning interact, and where does the messaging layer stop and CRDT/OT begin?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Breaking Schema Changes](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Fat_vs_Thin_Events/Breaking_Schema_Changes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary:

In this scenario, we need to architect a real-time collaborative editing backend system that supports thousands of concurrent users editing the same document in an event-driven manner. The key challenges are ensuring data consistency through ordering and conflict resolution mechanisms while leveraging the strengths of both message queues and conflict-free replicated data types (CRDTs). We will design a solution that integrates message queues for initial event propagation, CRDTs to handle conflicts, and partitioning strategies to manage scalability.

#### Triaging the Question

The question is probing your understanding of distributed systems architecture, specifically focusing on real-time collaborative editing. Key aspects include:

Message Queuing: Efficiently distributing events across nodes.
Conflict Resolution (CRDT/OT): Ensuring data consistency when multiple users edit the same document simultaneously.
Partitioning: Managing large numbers of concurrent users by dividing the workload.

#### Assumptions
Users are editing a document in real-time, meaning updates must be visible to all parties as soon as possible.
The system needs to handle thousands of simultaneous users without degradation in performance or data integrity.
Each user can make local edits and these edits need to be propagated to other peers.
Step-by-Step Solution:
1. Event Substrate

First, define the event-driven architecture where every user action (e.g., text insertion, deletion, drawing a line) is represented as an event. These events are lightweight JSON objects containing metadata such as user_id, timestamp, and action.

JSON
{
    "user_id": 123,
    "timestamp": "2023-10-05T14:36:00Z",
    "action": {
        "type": "insert_text",
        "data": {"position": [0, 0], "text": "Hello"}
    }
}
2. Message Queues

We will use a distributed message queue system to handle event propagation. RabbitMQ or Kafka are suitable choices because they provide features like reliability, scalability, and fault tolerance.

Queue Setup: One queue per document. When an edit is made by a user, the client sends the event to this queue.

#### Cons
umer Logic: Consumers (nodes) subscribe to these queues, consume events, process them using CRDTs, and then broadcast any necessary updates back into other relevant queues.
User Edits Document
Event Produced
Message Queue
Queue Consumer Node 1
Queue Consumer Node 2
Broadcast to Other Queues
Broadcast to Other Queues
3. CRDT/Operational Transformation (OT)

CRDTs and OT are used for conflict resolution because they can handle concurrent edits in a distributed environment without explicit coordination.

Conflict-Free Replicated Data Types: For example, SetCRDT or VectorClockCRDT can be used to manage sets of insertions/deletions.
Operational Transformation (OT): This is a method where local operations are transformed into remote ones so that any conflicts arising from concurrent edits can be resolved locally.
Local Operation
Turned into Remote Operation
Sent to Other Consumers
Transformed Back on ReceiverSide
4. Per-Document Partitioning

To manage thousands of users, we need to partition the document space.

Partition Key: Use a hash function that maps each user or document segment to a specific node.
Sharding Strategy: Distribute documents across nodes so that only relevant nodes handle updates and reads for those documents. This reduces contention and improves scalability.
User ID
Partition Key
Node Assignment
Node 1|Node 2|Node 3

#### Closing Summary

By combining message queues for initial event propagation, CRDTs and OT for conflict resolution, and partitioning to manage large-scale concurrent edits, we can build a robust real-time collaborative editing backend. This approach ensures high availability, scalability, and consistency across thousands of simultaneous users.

#### Common Mistakes/Follow-Ups
Ignoring Latency: Always consider network latency when designing event-driven architectures.
Handling Large Messages: Optimize message sizes to avoid bottlenecks in the queue system.
Scalability Considerations: Ensure that your CRDT implementation and partitioning strategy can handle high traffic without degrading performance.

**Comparison:**

```
Node	Document ID	User Count
N1	Doc1	500
N2	Doc2	700
```

**Code:**

```python
# Example Message Queue Producer (RabbitMQ)
import pika

def produce_event(queue_name, event):
    connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
    channel = connection.channel()
    channel.queue_declare(queue=queue_name)
    channel.basic_publish(exchange='', routing_key=queue_name, body=str(event))
    print(" [x] Sent %r" % event)
    connection.close()

# Example CRDT Update Function
class SetCRDT:
    def __init__(self):
        self.values = set()

    def insert(self, value):
        if not value in self.values:
            self.values.add(value)

    def remove(self, value):
        if value in self.values:
            self.values.remove(value)
```

**Example input / output:**

```text
Input:
User ID: 123
Action: Insert Text at Position [0, 0] with Text "Hello"

Output:
Document State Update:
- New Text: "Hello" inserted at position [0, 0]
```

**Data:**

```json
{
    "user_id": 123,
    "timestamp": "2023-10-05T14:36:00Z",
    "action": {
        "type": "insert_text",
        "data": {"position": [0, 0], "text": "Hello"}
    }
}
```

**Flow:**

1. Rendering diagram…

