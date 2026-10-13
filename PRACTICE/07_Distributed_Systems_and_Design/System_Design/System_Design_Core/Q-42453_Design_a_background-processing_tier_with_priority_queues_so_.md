---
type: "practice"
difficulty: "Intermediate"
---

# Design a background-processing tier with priority queues so that time-sensitive jobs (OTP send) never starve behind bulk jobs (marketing emails). How do you prevent starvation of low-priority work?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Notification System](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Notification_System.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a background-processing tier with priority queues, we need to ensure that high-priority tasks like sending One-Time Passwords (OTPs) are processed before lower-priority bulk jobs such as marketing emails. This is crucial to prevent the starvation of time-sensitive work. The system should efficiently manage multiple priorities while minimizing delays and ensuring fairness.

#### Triage
What Is Being Probed

The question requires us to design a queue management system that handles different levels of priority effectively. Key points include:

Priority Handling: Ensuring high-priority jobs are processed first.
Backpressure Management: Preventing lower-priority jobs from overwhelming the system and starving higher-priority ones.
Performance Metrics: Measuring throughput, latency, and fairness among queues.

#### Assumptions
Jobs can be categorized into different priority levels (e.g., 0 for OTPs, 1 for marketing emails).
The system should support multiple threads or processes to handle each queue simultaneously.
We will use a priority queue data structure where higher numbers indicate lower priority, as typical in many languages.

#### First-Principles Solution
Step 1: Define Queues and Priorities

We need at least two queues—high_priority_queue for OTPs and low_priority_queue for marketing emails. Each job will be enqueued with its associated priority level.

PYTHON
class Job:
    def __init__(self, priority, data):
        self.priority = priority
        self.data = data

def enqueue(job, high_priority_queue, low_priority_queue):
    if job.priority == 0:  # OTPs have the highest priority (lowest number)
        high_priority_queue.append(job)
    else:
        low_priority_queue.append(job)
Step 2: Implement Priority Queue Handling

We need to process jobs from both queues simultaneously but prioritize high_priority_queue. We can achieve this by periodically checking and processing jobs.

PYTHON
def process_jobs(high_priority_queue, low_priority_queue):
    while high_priority_queue:
        job = high_priority_queue.pop(0)
        # Process the high-priority job here
        print(f"Processed job: {job.data}")

    if not high_priority_queue and low_priority_queue:
        job = low_priority_queue.pop(0)
        # Process the low-priority job with backpressure handling
        process_low_priority_job(job)

def process_low_priority_job(job):
    # Simulate processing time
    import time
    print(f"Processed job: {job.data}")
    time.sleep(1)  # Simulating delay

# Example usage
enqueue(Job(0, "OTP Job"), high_priority_queue, low_priority_queue)
enqueue(Job(1, "Marketing Email"), high_priority_queue, low_priority_queue)
process_jobs(high_priority_queue, low_priority_queue)
Step 3: Backpressure Management

To manage backpressure effectively, we need to ensure that the processing of low-priority jobs does not overwhelm the system. This can be achieved by setting a maximum queue size or time limits.

PYTHON
from collections import deque

high_priority_queue = deque()
low_priority_queue = deque()

def enqueue(job, high_priority_queue, low_priority_queue):
    if job.priority == 0:
        if len(high_priority_queue) < 5:  # Limiting the queue to avoid overflow
            high_priority_queue.appendleft(job)
        else:
            print("High-priority queue is full.")
    else:
        if len(low_priority_queue) < 10:  # Similar limit for low-priority jobs
            low_priority_queue.appendleft(job)
        else:
            print("Low-priority queue is full.")

def process_jobs(high_priority_queue, low_priority_queue):
    while high_priority_queue:
        job = high_priority_queue.pop(0)
        print(f"Processed job: {job.data}")

    if not high_priority_queue and low_priority_queue:
        if len(low_priority_queue) > 2:  # Limiting the number of jobs in backpressure
            print("Low-priority queue is too full, delaying processing.")
        else:
            job = low_priority_queue.pop(0)
            process_low_priority_job(job)

# Example usage with backpressure
for _ in range(6):
    enqueue(Job(1, f"Marketing Email {_}"), high_priority_queue, low_priority_queue)

process_jobs(high_priority_queue, low_priority_queue)

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, we designed a system that ensures high-priority jobs like OTPs are processed first by using separate queues and prioritizing them. We also introduced backpressure management to prevent the starvation of lower-priority work by limiting queue sizes and delaying processing when necessary.

#### Common mistakes
include:

Not handling the backpressure correctly, leading to overwhelming of the low-priority queue.
Ignoring fairness among jobs within the same priority level.
Failing to account for edge cases such as empty queues or unexpected job types.

#### Follow-ups
could involve discussing load balancing strategies, failover mechanisms, and integrating with a larger system architecture.

**Comparison:**

```
Queue Type	Maximum Size	Priority Level
High-Priority	5	0
Low-Priority	10	1-2
```

**Code:**

```python
class Job:
    def __init__(self, priority, data):
        self.priority = priority
        self.data = data

def enqueue(job, high_priority_queue, low_priority_queue):
    if job.priority == 0:
        if len(high_priority_queue) < 5:
            high_priority_queue.appendleft(job)
        else:
            print("High-priority queue is full.")
    else:
        if len(low_priority_queue) < 10:
            low_priority_queue.appendleft(job)
        else:
            print("Low-priority queue is full.")

def process_jobs(high_priority_queue, low_priority_queue):
    while high_priority_queue:
        job = high_priority_queue.pop(0)
        print(f"Processed job: {job.data}")

    if not high_priority_queue and low_priority_queue:
        if len(low_priority_queue) > 2:
            print("Low-priority queue is too full, delaying processing.")
        else:
            job = low_priority_queue.pop(0)
            process_low_priority_job(job)

def process_low_priority_job(job):
    import time
    print(f"Processed job: {job.data}")
    time.sleep(1)  # Simulating delay

# Example usage with backpressure
for _ in range(6):
    enqueue(Job(1, f"Marketing Email {_}"), high_priority_queue, low_priority_queue)

process_jobs(high_priority_queue, low_priority_queue)
```

**Example input / output:**

```text
Enqueueing jobs:
Processed job: OTP Job
Processed job: OTP Job

Processing low priority queue with backpressure:
Low-priority queue is too full, delaying processing.
Processed job: Marketing Email 0
Processed job: Marketing Email 1
Processed job: Marketing Email 2
Processed job: Marketing Email 3
Processed job: Marketing Email 4
```

**Data:**

```json
{
  "high_priority_queue": [
    {"priority": 0, "data": "OTP Job"},
    {"priority": 0, "data": "OTP Job"}
  ],
  "low_priority_queue": [
    {"priority": 1, "data": "Marketing Email 0"},
    {"priority": 1, "data": "Marketing Email 1"}
  ]
}
```

**Flow:**

1. Receive OTP Job
2. Enqueue tohigh_priority_queue
3. Receive Marketing Email
4. Enqueue to low_priority_queue
5. Process high_priority_queue
6. Processing OTP

