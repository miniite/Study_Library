---
type: "practice"
difficulty: "Intermediate"
---

# Explain the semantics of a semaphore vs a mutex vs a latch. Give a backend use case for each (throttling downstream calls, mutual exclusion, one-time startup gate) and a bug from confusing them.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Bulkheads and Thread Pools](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Bulkheads_and_Thread_Pools.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In concurrency control, semaphores, mutexes, and latches are all fundamental constructs used to manage access to shared resources, but they serve different purposes and have distinct semantics. A semaphore is used to signal a count of available items in a resource pool, allowing multiple threads to access the resource while ensuring that the number of simultaneous accesses does not exceed a limit. A mutex (mutual exclusion) is designed for locking a single resource to ensure mutual exclusion—only one thread can access it at any time. Finally, a latch serves as a simple barrier mechanism, where all threads wait until a specific condition is met.

Let's explore the semantics and use cases of each:

Semaphores

Semaphores are often used in scenarios like throttling downstream API calls to avoid overwhelming them. For example, if you have an application that makes multiple requests to a slow or rate-limited service, using a semaphore with a limited number of permits can ensure that only a certain number of concurrent requests are made at any time.

Mutexes

Mutexes are used for ensuring mutual exclusion when accessing shared resources. In a backend system, this might be applied in a database connection pool, where each thread needs to acquire a mutex before using the connection and release it afterward to prevent multiple threads from stepping on each other's toes.

Latches

Latches are often employed as one-time startup gates. For example, when initializing a distributed service, you want all instances of that service to wait until they receive confirmation that the critical section is ready before proceeding with their work. This ensures that every instance starts in the same state.

Now let's look at some common bugs that can arise from confusing these constructs:

Semaphore vs Mutex Bug:

Scenario: You have a rate-limited API call that should be throttled, but instead of using a semaphore to control the number of concurrent calls, you mistakenly use a mutex.
Issue: This will limit only one thread at a time from making an API call, which is not what you want. Instead, multiple threads would wait in line behind each other, effectively serializing all requests and defeating the purpose of rate limiting.

Mutex vs Latch Bug:

Scenario: In a distributed system, you need to ensure that all nodes start up only after a leader has been elected.
Issue: If you mistakenly use a mutex instead of a latch, threads might continuously try to acquire the lock, causing them to spin or block indefinitely. A latch would allow multiple threads to wait until the election is complete and then proceed.

Semaphore vs Latch Bug:

Scenario: You want to throttle incoming requests to an API but mistakenly use a latch.
Issue: If you use a latch, all threads will wait until the latch condition becomes true (which might never happen in a non-blocking scenario), or they may proceed too early if the latch is misused.
Backend Use Cases
Semaphore
Throttling API Calls:
PYTHON
from threading import BoundedSemaphore

semaphore = BoundedSemaphore(value=5)  # Allow only 5 concurrent requests at a time.

def make_api_call():
    with semaphore:
        # Make the API call here...
        pass
Mutex
Database Connection Pool:
PYTHON
from threading import Lock

connection_pool_lock = Lock()

def get_database_connection():
    with connection_pool_lock:
        # Acquire a database connection and use it.
        pass
Latch
One-Time Startup Gate for Distributed Services:
PYTHON
from threading import Event, Thread

startup_event = Event()

def startup_gate():
    print("Waiting until the startup event is set.")
    startup_event.wait()  # Wait until the event becomes true.
    print("Startup completed. Proceeding with initialization.")

for _ in range(3):
    thread = Thread(target=startup_gate)
    thread.start()

import time
time.sleep(5)  # Simulate delay before setting the event.
startup_event.set()  # All threads will now proceed.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, semaphores, mutexes, and latches each serve specific purposes in managing concurrent access to shared resources. Semaphores are used for rate limiting and resource pooling, mutexes ensure mutual exclusion, and latches act as one-time startup gates.

#### Common mistakes
include misusing a semaphore as a mutex or vice versa, using a latch when a semaphore is needed, or misunderstanding the behavior of a latch in distributed systems. Always ensure you choose the correct concurrency control mechanism based on your specific use case to avoid bugs and performance issues.

For follow-ups:

What are other concurrency patterns commonly used in backend development?
How would you implement these patterns in a multi-threaded environment?

**Code:**

```python
from threading import BoundedSemaphore, Lock, Event

# Semaphore example
semaphore = BoundedSemaphore(value=5)

def make_api_call():
    with semaphore:
        # Make the API call here...
        pass

# Mutex example
connection_pool_lock = Lock()

def get_database_connection():
    with connection_pool_lock:
        # Acquire a database connection and use it.
        pass

# Latch example
startup_event = Event()

def startup_gate():
    print("Waiting until the startup event is set.")
    startup_event.wait()  # Wait until the event becomes true.
    print("Startup completed. Proceeding with initialization.")

for _ in range(3):
    thread = Thread(target=startup_gate)
    thread.start()

import time
time.sleep(5)  # Simulate delay before setting the event.
startup_event.set()
```

**Flow:**

1. Throttle API Calls
2. Semaphore
3. Distribute DB Connections
4. Lock
5. Startup Gate
6. Event

