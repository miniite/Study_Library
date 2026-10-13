---
type: "practice"
difficulty: "Deep"
---

# A queue consumer must process messages in per-key order but across keys in parallel; how do you design the partition-to-worker mapping so that ordering holds without one slow key blocking all others?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Bounded Blocking Queues](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Bounded_Blocking_Queues.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this problem, we need to ensure that messages are processed in per-key order but can be handled concurrently across different keys. This requires a carefully designed partition-to-worker mapping strategy. The core idea is to split the work into partitions based on message keys and then assign these partitions to workers such that each worker processes its own key group in sequence while allowing parallel processing among different keys.

#### Triage

The interviewer is probing for an understanding of how to manage concurrent execution with order constraints, ensuring that messages are processed in a specific order (per-key) but leveraging the benefits of parallelism across multiple keys. Key assumptions include:

Each message has a unique key.
There can be varying numbers of workers and keys.
The goal is to avoid any single slow key from blocking processing for other keys.

#### First Principles Solution
Message Queue Partitioning: Split the queue into partitions based on the message keys. Each partition contains messages belonging to the same key but in their original order.
Worker Assignment: Assign each worker a unique partition (or set of partitions) such that no two workers are assigned overlapping partitions. Workers should process their assigned partitions sequentially.
Detailed Steps

Queue Partitioning:

Use a hash function on the message key to determine which partition a message belongs to.
Ensure the distribution is even across partitions to avoid some workers being overburdened while others are idle.

Worker Assignment and Scheduling:

Assign workers based on available slots in each partition. Each worker should be assigned a unique set of partitions, ensuring no overlap.
Use a round-robin or priority-based scheduling mechanism for assigning workers to specific partitions as they become free.

Concurrency Management:

Use thread pools or task queues within each partition to manage concurrent processing efficiently.
Implement proper synchronization mechanisms (locks, semaphores) at the partition level to ensure messages are processed in order while allowing parallel execution across different keys.

Failure and Recovery:

Implement checkpointing and recovery mechanisms to handle worker failures. Ensure that when a worker fails, another worker can take over its partitions without disrupting the processing order.
Use heartbeats or periodic check-ins between workers and a central manager to ensure all workers are aware of their current state.

#### Closing Summary

By carefully partitioning the queue based on message keys and assigning unique partitions to each worker, we can achieve both sequential per-key processing and parallelism across different keys. This design ensures that slow-performing keys do not block other faster ones, optimizing overall throughput while maintaining the required order constraints.

#### Common Mistakes/Follow-Ups
Ignoring Key Distribution: Failing to properly distribute key partitions evenly among workers can lead to imbalanced workloads.
Lack of Synchronization: Poorly managed concurrency can result in race conditions or incorrect message ordering within a partition.
Insufficient Failure Handling: Not implementing robust recovery mechanisms can cause data loss or processing inconsistencies if a worker fails.

**Comparison:**

| Worker ID | Assigned Partitions |
|-----------|---------------------|
| 0         | [Partition A]       |
| 1         | [Partition B]       |

**Code:**

```python
# Example code snippet for assigning workers to partitions

from collections import defaultdict
import threading

class QueueConsumer:
    def __init__(self, num_workers):
        self.partitions = defaultdict(list)
        self.workers = [threading.Thread(target=self.process_partition, args=(i,)) for i in range(num_workers)]

    def assign_partitions(self, messages):
        """Assign partitions based on message keys."""
        for msg in messages:
            key_hash = hash(msg.key) % num_partitions
            self.partitions[key_hash].append(msg)

    def start(self):
        """Start all worker threads."""
        for worker in self.workers:
            worker.start()

    def process_partition(self, worker_id):
        """Process assigned partitions sequentially."""
        for partition in self.partitions:
            if worker_id == partition:  # Simulate unique assignment
                with threading.Lock():  # Synchronization mechanism
                    for msg in self.partitions[partition]:
                        self.consume_message(msg)

    def consume_message(self, message):
        """Process a single message."""
        print(f"Worker {worker_id} processing {message}")

# Example usage
messages = [{"key": "A", "content": "Message 1"}, {"key": "B", "content": "Message 2"}]
consumer = QueueConsumer(2)
consumer.assign_partitions(messages)
consumer.start()
```

**Example input / output:**

```text
No specific input-output example needed for this conceptual solution, but the code snippet above demonstrates the flow.
```

**Data:**

```json
[
    {"key": "A", "content": "Message 1"},
    {"key": "B", "content": "Message 2"}
]
```

**Flow:**

1. Queue Consumer Starts
2. Assign Partitions
3. Assign Messages to Partitions
4. Start Workers
5. Worker 0 Starts ProcessingPartition A
6. Worker 1 Starts ProcessingPartition B

