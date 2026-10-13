---
type: "practice"
difficulty: "Expert"
---

# You need per-key serialization (all operations for user X run in order, but different users run in parallel). Design a keyed-executor / actor-per-key pattern and discuss how you bound memory when keys are unbounded.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Bulkheads and Thread Pools](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Bulkheads_and_Thread_Pools.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To tackle the requirement of per-key serialization (ensuring that all operations on user X run sequentially but allowing parallelism across different users), we need to design a pattern known as a "keyed-executor" or an "actor-per-key" approach. This involves creating separate executors or actors for each key, ensuring that all operations related to the same key are processed in order while multiple keys can be handled concurrently.

#### Triaging the Question

The core of this question is about designing a concurrency pattern that supports per-key serialization while efficiently managing memory usage when the number of keys (users) is unbounded. The interviewer likely wants to assess your understanding of concurrency patterns, race conditions, and how to manage state and resources in a scalable manner.

#### First-Principles Solution
Keyed Executor Pattern: Create an executor or actor for each key. This ensures that operations on the same key are processed sequentially.
Concurrency Control: Use synchronization mechanisms (e.g., semaphores, locks) to manage concurrent access within each key’s context.
Memory Management: Since keys can be unbounded, we need a strategy to ensure memory efficiency without compromising performance.
Detailed Solution

Keyed Executor Design:

Each key gets its own thread or task executor.
Operations on the same key are placed into a queue and processed sequentially by that key's executor.

Concurrency Control:

Use semaphores or other synchronization primitives to manage concurrent access within each key’s context, ensuring operations on the same key do not interfere with each other.

Memory Management:

Since we cannot predict the number of keys, a per-key executor approach must be memory-efficient.
Leverage thread pools where each thread is associated with a specific key to share common resources while keeping per-key state isolated.

#### Example
Code
PYTHON
import threading

class KeyedExecutor:
    def __init__(self):
        self.key_executors = {}

    def execute(self, key, operation):
        if key not in self.key_executors:
            # Create a new executor for this key
            self.key_executors[key] = threading.Thread(target=self.run_operations, args=(key,))
            self.key_executors[key].start()

        # Add the operation to the key's queue and let the executor handle it
        self.key_executors[key].queue.put(operation)

    def run_operations(self, key):
        while True:
            operation = self.key_executors[key].queue.get()
            if operation is None:
                break
            # Process the operation
            print(f"Processing {operation} for key {key}")
            # Simulate processing time
            import time
            time.sleep(1)
            self.key_executors[key].queue.task_done()

# Example usage
executor = KeyedExecutor()
executor.execute('user1', 'task1')
executor.execute('user2', 'task2')
executor.execute('user1', 'task3')

for key in executor.key_executors:
    executor.key_executors[key].queue.put(None)

#### Closing Summary
+ Common Mistakes
Memory Bloating: One common mistake is not managing memory efficiently, leading to high memory usage as the number of keys grows.
Concurrency Issues: Another pitfall is improper handling of concurrency within each key’s context, which can lead to race conditions and deadlocks.
Resource Contention: Ensuring that operations on different keys do not contend for shared resources while maintaining per-key isolation is crucial.
Common Follow-Ups
How would you handle memory constraints if the number of keys significantly increases?
Can you optimize the design further to reduce memory overhead?
What happens in case of a key being deleted or an executor failure?

**Code:**

```python
import threading

class KeyedExecutor:
    def __init__(self):
        self.key_executors = {}

    def execute(self, key, operation):
        if key not in self.key_executors:
            # Create a new executor for this key
            self.key_executors[key] = threading.Thread(target=self.run_operations, args=(key,))
            self.key_executors[key].start()

        # Add the operation to the key's queue and let the executor handle it
        self.key_executors[key].queue.put(operation)

    def run_operations(self, key):
        while True:
            operation = self.key_executors[key].queue.get()
            if operation is None:
                break
            # Process the operation
            print(f"Processing {operation} for key {key}")
            # Simulate processing time
            import time
            time.sleep(1)
            self.key_executors[key].queue.task_done()

# Example usage
executor = KeyedExecutor()
executor.execute('user1', 'task1')
executor.execute('user2', 'task2')
executor.execute('user1', 'task3')

for key in executor.key_executors:
    executor.key_executors[key].queue.put(None)
```

