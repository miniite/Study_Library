---
type: "practice"
difficulty: "Deep"
---

# Explain the difference between a data race and a race condition. Give a concrete example where code has a race condition but no data race, and one with a data race but arguably correct output.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Concurrent Data Structures](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Concurrent_Data_Structures.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In concurrent programming, understanding concurrency patterns and race conditions is crucial to ensure robustness and correctness in multi-threaded applications. A race condition occurs when the behavior of the program depends on the sequence or timing of uncontrollable events, such as thread scheduling or execution order. In contrast, a data race specifically happens when multiple threads access shared data simultaneously, with at least one thread writing to the data without proper synchronization.

#### Triaging the Question

The interviewer is probing your understanding of fundamental concurrency issues and the ability to illustrate subtle differences between race conditions and data races through practical examples. You need to demonstrate a deep understanding by providing concrete scenarios that highlight these distinctions clearly.

#### Example
1: Race Condition Without Data Race

#### Cons
ider a simple counter increment operation without explicit synchronization:

PYTHON
# Example 1: Counter with no sync, but potential race condition
counter = [0]

def increment():
    for _ in range(100):
        counter[0] += 1

threads = []
for i in range(5):
    thread = threading.Thread(target=increment)
    threads.append(thread)
    thread.start()

# Wait for all threads to finish
for thread in threads:
    thread.join()

print("Final value:", counter[0])

In this example, the counter variable is accessed and modified by multiple threads. The behavior of the program could vary based on how threads are scheduled, leading to a race condition where the final value of counter might not be 500 due to interleaved execution.

#### Example
2: Data Race but Correct Output

Now let's consider an example with data races but still producing the correct output:

PYTHON
# Example 2: Counter with non-blocking writes, but no data race (arguably)
import threading

counter = [0]
lock = threading.Lock()

def increment():
    for _ in range(100):
        # Simulate atomic write by using a lock to guard against simultaneous writes
        if counter[0] < 500:
            with lock:
                counter[0] += 1

threads = []
for i in range(5):
    thread = threading.Thread(target=increment)
    threads.append(thread)
    thread.start()

# Wait for all threads to finish
for thread in threads:
    thread.join()

print("Final value:", counter[0])

In this case, the counter variable is updated with a lock, which means only one thread can write at a time. This avoids data races but still allows multiple threads to read from the same memory location simultaneously. However, since updates are guarded by the lock, the final output will be correct regardless of the interleaved execution.

#### Closing Summary
and Common Mistakes/Follow-Ups

The key difference is that a race condition can occur without a data race if shared resources are not directly modified concurrently. Conversely, a data race implies concurrent access to mutable state without proper synchronization mechanisms.

#### Common Mistakes

Overlooking Synchronization: Failing to recognize the need for locks or other synchronization primitives.
Misunderstanding Sequence Dependence: Assuming that order of operations is guaranteed and not considering thread scheduling nuances.
Ignoring Lock Granularity: Not understanding how lock granularity affects race conditions and data races.

During follow-ups, you might be asked about potential solutions, such as using higher-level concurrency constructs (e.g., threading.Condition, asyncio), or exploring more sophisticated synchronization strategies like read-write locks or atomic operations.

**Comparison:**

```
Example	Description
1	Race Condition Without Data Race
2	Data Race but Correct Output
```

**Code:**

```python
# Example 1: Counter with no sync, but potential race condition
counter = [0]

def increment():
    for _ in range(100):
        counter[0] += 1

threads = []
for i in range(5):
    thread = threading.Thread(target=increment)
    threads.append(thread)
    thread.start()

# Wait for all threads to finish
for thread in threads:
    thread.join()

print("Final value:", counter[0])

# Example 2: Counter with non-blocking writes, but no data race (arguably)
import threading

counter = [0]
lock = threading.Lock()

def increment():
    for _ in range(100):
        # Simulate atomic write by using a lock to guard against simultaneous writes
        if counter[0] < 500:
            with lock:
                counter[0] += 1

threads = []
for i in range(5):
    thread = threading.Thread(target=increment)
    threads.append(thread)
    thread.start()

# Wait for all threads to finish
for thread in threads:
    thread.join()

print("Final value:", counter[0])
```

**Example input / output:**

```

import threading

counter = [0]

def increment(): ... for _ in range(100): ... counter[0] += 1

threads = [] for i in range(5): ... thread = threading.Thread(target=increment) ... threads.append(thread) ... thread.start()

for thread in threads: ... thread.join()

print("Final value:", counter[0])

CODE
Copy

```python
>>> import threading

>>> counter = [0]
>>> lock = threading.Lock()

>>> def increment():
...     for _ in range(100):
...         if counter[0] < 500:
...             with lock:
...                 counter[0] += 1

>>> threads = []
>>> for i in range(5):
...     thread = threading.Thread(target=increment)
...     threads.append(thread)
...     thread.start()

>>> for thread in threads:
...     thread.join()

>>> print("Final value:", counter[0])
```

**Data:**

```json
{
  "counter": [0]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Thread 1
6. Is counter < 500?
7. Increment counter
8. Cycle continues
9. Thread 2
10. Is counter < 500?
11. Increment counter
12. Cycle continues
13. Final value: 496

