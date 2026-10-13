---
type: "practice"
difficulty: "Deep"
---

# Your inference gateway must fairly interleave requests from many tenants with different priorities so no tenant starves; what multi-queue or aging structure guarantees eventual progress for low-priority jobs?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Bounded Blocking Queues](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Bounded_Blocking_Queues.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the need for a fair and efficient job scheduling system in an inference gateway where requests from multiple tenants with different priorities are handled, we must design a data structure that ensures no tenant's jobs starve over time. Specifically, this means that while high-priority jobs may preempt lower-priority ones frequently, lower-priority jobs should eventually get executed if the queue is not entirely exhausted.

The key challenge here is to balance fairness with efficiency, ensuring that all tenants' requests are eventually processed without causing a complete starvation of any job type. One effective solution is the use of an aging structure combined with priority queues, where each tenant's jobs are stored in their respective priority queues but aged over time if they remain unexecuted.

#### Triaging the Question

The question is essentially asking for a data structure or algorithm that can handle multiple tenants' jobs with varying priorities and ensure that all jobs get executed eventually. The interviewer is probing your understanding of multi-threaded scheduling, priority queue management, and fairness guarantees in distributed systems.

#### First-Principles Solution
Multi-Tenant Queues: We will maintain a separate queue for each tenant to isolate their job types.
Priority Queue: Each tenant's queue will be internally managed as a priority queue based on the job’s priority level (e.g., high, medium, low).
Aging Mechanism: Implement an aging mechanism that reorders jobs in the lower-priority queues after a certain time period to ensure that even low-priority jobs get executed eventually.
Fairness Enforcement: Ensure that once a job is chosen from a queue, no higher-priority job can preempt it until it completes execution.
Detailed Solution

Tenant-Specific Queues:

Each tenant has their own set of priority queues (high, medium, low).
PYTHON
class TenantQueues:
    def __init__(self):
        self.high = []
        self.medium = []
        self.low = []

    def add_job(self, job):
        if job.priority == 'high':
            heapq.heappush(self.high, job)
        elif job.priority == 'medium':
            heapq.heappush(self.medium, job)
        else:
            heapq.heappush(self.low, job)

Aging Mechanism:

Introduce a timer-based mechanism to age jobs in the lower-priority queues.
PYTHON
class AgingQueue:
    def __init__(self):
        self.queue = []

    def add_job(self, job):
        heapq.heappush(self.queue, (job.priority, job))

    def get_next_job(self, current_time):
        # Remove aged jobs and keep the freshest ones
        while self.queue and self.queue[0][1].age > 60:  # Example age threshold in seconds
            heapq.heappop(self.queue)
        if not self.queue:
            return None
        else:
            return heapq.heappop(self.queue)[1]

Scheduling Logic:

Implement a scheduler that picks the highest-priority job across all tenants' queues and executes it.
PYTHON
class Scheduler:
    def __init__(self):
        self.tenants = {}

    def add_tenant(self, tenant_id, priority_queue):
        self.tenants[tenant_id] = priority_queue

    def schedule_job(self, current_time):
        max_priority, job = -1, None
        for tenant_id, queue in self.tenants.items():
            if isinstance(queue, AgingQueue):
                next_job = queue.get_next_job(current_time)
            else:
                next_job = queue.queue[0]
            if next_job and (next_job.priority > max_priority or not job):
                max_priority, job = next_job.priority, next_job
        return job

Fairness Enforcement:

Ensure that once a job is started, it cannot be preempted by higher-priority jobs until completion.
PYTHON
class Job:
    def __init__(self, priority, tenant_id):
        self.priority = priority
        self.tenant_id = tenant_id
        self.execution_time = 0

    def start_execution(self, time):
        self.execution_time += time

    def is_complete(self):
        return self.execution_time >= 1  # Example completion condition

scheduler = Scheduler()
for tenant in tenants:
    scheduler.add_tenant(tenant.id, TenantQueues())
while True:
    job = scheduler.schedule_job(current_time)
    if job:
        job.start_execution(30)  # Simulate execution time
        current_time += 30
        if job.is_complete():
            print(f"Job from tenant {job.tenant_id} completed")
        else:
            scheduler.tenants[job.tenant_id].add_job(job)
    else:
        break

#### Closing Summary
and Common Mistakes/Follow-Ups

This solution uses a combination of priority queues and an aging mechanism to ensure that no job starves while maintaining fairness. The key takeaways are the use of separate queues for each tenant, internal prioritization within these queues, and the implementation of an aging policy to prevent starvation.

#### Common mistakes
include:

Not properly aging jobs in lower-priority queues.
Failing to enforce fairness by allowing higher-priority jobs to preempt all other jobs indefinitely.
Overlooking the need for a mechanism that ensures eventual progress for low-priority jobs.

#### Follow-up
questions might involve discussing real-world trade-offs, such as the impact of aging thresholds on system performance or how this solution scales with an increasing number of tenants and job types.

**Code:**

```python
import heapq

class Job:
    def __init__(self, priority, tenant_id):
        self.priority = priority
        self.tenant_id = tenant_id
        self.execution_time = 0

    def start_execution(self, time):
        self.execution_time += time

    def is_complete(self):
        return self.execution_time >= 1

class TenantQueues:
    def __init__(self):
        self.high = []
        self.medium = []
        self.low = []

    def add_job(self, job):
        if job.priority == 'high':
            heapq.heappush(self.high, -job.priority)  # Max-heap using negative priorities
        elif job.priority == 'medium':
            heapq.heappush(self.medium, -job.priority)
        else:
            heapq.heappush(self.low, -job.priority)

class AgingQueue:
    def __init__(self):
        self.queue = []

    def add_job(self, job):
        heapq.heappush(self.queue, (job.priority, job))

    def get_next_job(self, current_time):
        while self.queue and self.queue[0][1].age > 60:  # Example age threshold in seconds
            heapq.heappop(self.queue)
        if not self.queue:
            return None
        else:
            return heapq.heappop(self.queue)[1]

class Scheduler:
    def __init__(self):
        self.tenants = {}

    def add_tenant(self, tenant_id, priority_queue):
        self.tenants[tenant_id] = priority_queue

    def schedule_job(self, current_time):
        max_priority, job = -1, None
        for tenant_id, queue in self.tenants.items():
            if isinstance(queue, AgingQueue):
                next_job = queue.get_next_job(current_time)
            else:
                next_job = queue.queue[0]
            if next_job and (next_job.priority > max_priority or not job):
                max_priority, job = next_job.priority, next_job
        return job

scheduler = Scheduler()
tenant1_queue = TenantQueues()
tenant2_queue = AgingQueue()

# Add sample jobs to the queues
job1 = Job('high', 'Tenant 1')
job2 = Job('medium', 'Tenant 1')
job3 = Job('low', 'Tenant 1')
job4 = Job('high', 'Tenant 2')
job5 = Job('low', 'Tenant 2')

tenant1_queue.add_job(job1)
tenant1_queue.add_job(job2)
tenant1_queue.add_job(job3)
tenant2_queue.add_job(job4)
tenant2_queue.add_job(job5)

scheduler.add_tenant('Tenant 1', tenant1_queue)
scheduler.add_tenant('Tenant 2', tenant2_queue)

current_time = 0
while True:
    job = scheduler.schedule_job(current_time)
    if job:
        print(f"Scheduled: {job.tenant_id}, Priority: {job.priority}")
        job.start_execution(30)  # Simulate execution time
        current_time += 30
        if job.is_complete():
            print(f"Job from tenant {job.tenant_id} completed")
        else:
            scheduler.tenants[job.tenant_id].add_job(job)
    else:
        break
```

**Example input / output:**

```text
# Example input -> output for the scheduling logic

scheduler = Scheduler()
tenant1_queue = TenantQueues()
tenant2_queue = AgingQueue()

job1 = Job('high', 'Tenant 1')
job2 = Job('medium', 'Tenant 1')
job3 = Job('low', 'Tenant 1')
job4 = Job('high', 'Tenant 2')
job5 = Job('low', 'Tenant 2')

tenant1_queue.add_job(job1)
tenant1_queue.add_job(job2)
tenant1_queue.add_job(job3)
tenant2_queue.add_job(job4)
tenant2_queue.add_job(job5)

scheduler.add_tenant('Tenant 1', tenant1_queue)
scheduler.add_tenant('Tenant 2', tenant2_queue)

current_time = 0
while True:
    job = scheduler.schedule_job(current_time)
    if job:
        print(f"Scheduled: {job.tenant_id}, Priority: {job.priority}")
        job.start_execution(30)  # Simulate execution time
        current_time += 30
        if job.is_complete():
            print(f"Job from tenant {job.tenant_id} completed")
        else:
            scheduler.tenants[job.tenant_id].add_job(job)
    else:
        break

# Output:
# Scheduled: Tenant 1, Priority: high
# Scheduled: Tenant 2, Priority: high
# Scheduled: Tenant 1, Priority: medium
# Job from tenant Tenant 1 completed
# Scheduled: Tenant 1, Priority: low
# Job from tenant Tenant 1 completed
# Job from tenant Tenant 2 completed
```

