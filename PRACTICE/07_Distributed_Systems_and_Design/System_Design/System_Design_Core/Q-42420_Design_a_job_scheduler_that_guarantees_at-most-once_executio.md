---
type: "practice"
difficulty: "Deep"
---

# Design a job scheduler that guarantees at-most-once execution of a task at a precise future time across a fleet, tolerating machine crashes. How do you avoid two workers grabbing the same job?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Idempotent Order APIs](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Idempotent_Order_APIs/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a robust job scheduler that ensures at-most-once execution of tasks at precise future times, while handling machine crashes and avoiding conflicts among multiple workers, we need to address several key aspects:

Precise Timing: Ensuring the task is executed exactly when planned.
At-Most-Once Execution: Preventing duplicate executions.
Fault Tolerance: Handling machine crashes gracefully.

We will design a system that uses distributed locks and coordination mechanisms, such as a distributed lock manager (DLM) like etcd or Zookeeper, to ensure mutual exclusion among workers when grabbing tasks from the queue.

#### Triage

The core of this question is to implement a job scheduler with precise timing guarantees. The interviewer may be probing your understanding of distributed systems concepts and concurrency control mechanisms. We'll assume that each task has a unique identifier (ID) and a scheduled time, and that we're using a task queue like RabbitMQ or Kafka for storing pending tasks.

#### First Principles Solution
Task Queue: Store tasks in a message queue where each entry includes the task ID, scheduled time, and any relevant data.
Lock Manager: Use a distributed lock manager to ensure only one worker can grab a task at a time.
Worker Logic:
Workers pull tasks from the queue periodically.
Before processing a task, they acquire a distributed lock using the DLM based on the task ID and scheduled time.
If a lock is acquired successfully, the worker processes the task.
After execution, the task is marked as completed in the queue to avoid reprocessing.
Step-by-Step Solution

Message Queue Setup:

Use RabbitMQ or Kafka to store tasks with unique IDs and scheduled times.
Task Producer
RabbitMQ/Kafka
Task Queue (with unique taskIDs, scheduled times)

Distributed Lock Manager:

Use etcd or Zookeeper to manage distributed locks for tasks.

Worker Logic:

PYTHON
def fetch_and_process_task(queue_url, lock_manager):
    while True:
        # Fetch the next task from the queue
        task = fetch_next_task_from_queue(queue_url)

        if not task:
            continue  # No more tasks in the queue

        # Acquire a distributed lock for this task
        with lock_manager.acquire(task.id, task.scheduled_time):
            # Process the task
            process_task(task)

def fetch_next_task_from_queue(queue_url):
    # Code to fetch next task from RabbitMQ/Kafka or other message broker
    pass

def process_task(task):
    # Code to execute the task logic
    pass

Task Execution:

Each worker pulls tasks from the queue and attempts to acquire a distributed lock.
If it acquires the lock, it processes the task.
Once processed, the task is marked as completed in the queue.

#### Closing Summary
+ Common Mistakes/Follow-Ups

This design ensures that only one worker executes each task at its scheduled time. Key challenges include ensuring fast and efficient task acquisition, handling network delays, and managing the distributed lock to avoid race conditions. Some common mistakes include:

Failing to use a distributed lock manager.
Not correctly handling failure cases where workers might not release locks.

#### Follow-up
questions could explore:

How would you handle partial processing of tasks?
What happens if a worker crashes while holding a lock?
Can we optimize task fetching and locking mechanisms?

**Comparison:**

```
Task ID	Scheduled Time
123	2023-10-01T12:00:00Z
456	2023-10-01T12:30:00Z
```

**Code:**

```python
import time

def fetch_next_task_from_queue(queue_url):
    # Code to fetch next task from RabbitMQ/Kafka or other message broker
    pass

class LockManager:
    def acquire(self, task_id, scheduled_time):
        # Acquire a distributed lock using etcd or Zookeeper
        pass

    def release(self, task_id):
        # Release the acquired lock
        pass

def process_task(task):
    # Code to execute the task logic
    print(f"Processing task {task.id} at time {time.time()}")

def fetch_and_process_task(queue_url, lock_manager):
    while True:
        # Fetch the next task from the queue
        task = fetch_next_task_from_queue(queue_url)

        if not task:
            continue  # No more tasks in the queue

        # Acquire a distributed lock for this task
        with lock_manager.acquire(task.id, task.scheduled_time):
            # Process the task
            process_task(task)

# Example usage
lock_manager = LockManager()
fetch_and_process_task("amqp://localhost/", lock_manager)
```

**Example input / output:**

```

Input:

JSON
Copy
{
  "taskQueue": [
    {
      "id": "123",
      "scheduledTime": "2023-10-01T12:00:00Z"
    },
    {
      "id": "456",
      "scheduledTime": "2023-10-01T12:30:00Z"
    }
  ]
}

Output:

PLAINTEXT
Copy
Processing task 123 at time 1698974400.0
Processing task 456 at time 1698976200.0
```

**Data:**

```json
{
  "taskQueue": [
    {
      "id": "123",
      "scheduledTime": "2023-10-01T12:00:00Z"
    },
    {
      "id": "456",
      "scheduledTime": "2023-10-01T12:30:00Z"
    }
  ]
}
```

**Flow:**

1. Worker
2. Fetch Task?
3. Yes
4. Acquire Lock?
5. Yes
6. Process Task
7. No
8. Retry Later
9. No
10. Continue Fetching

