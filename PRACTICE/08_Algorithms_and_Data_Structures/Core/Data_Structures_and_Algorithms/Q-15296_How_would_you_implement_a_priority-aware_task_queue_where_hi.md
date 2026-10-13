---
type: "practice"
difficulty: "Deep"
---

# How would you implement a priority-aware task queue where high-priority jobs must not starve low-priority ones forever; what aging or multi-queue structure gives you fairness?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Bounded Blocking Queues](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Bounded_Blocking_Queues.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To implement a priority-aware task queue with fairness, we need to ensure that high-priority tasks are processed in a timely manner without starving low-priority tasks forever. A common approach is to use multiple queues or an aging mechanism to balance the priorities while maintaining fairness.

#### Triaging the Question

The core of this question lies in designing a system where:

Fairness: Lower priority tasks should not be indefinitely starved by higher priority ones.
Efficiency: High-priority tasks must receive timely processing.

We'll consider two primary strategies: using multiple queues and implementing an aging mechanism. Each approach has its trade-offs, which we'll explore in detail.

#### Assumptions
We're dealing with a multi-threaded or asynchronous environment where tasks can be processed concurrently.
The task queue is part of a larger system that processes various types of jobs (e.g., background processing, real-time data handling).
Priorities are represented as integer values, with higher numbers indicating higher priority.

#### First-Principles Solution
Multiple Queues Approach

One straightforward approach is to use multiple queues, each dedicated to a specific priority level or range of priorities. This ensures that tasks are processed in order but may lead to inefficiencies if the number of queues grows too large.

#### Implementation

Queue Setup: Create separate queues for different ranges of priorities.
Task Assignment: Assign new tasks to the appropriate queue based on their priority level.
Processing Logic: Process each queue in a round-robin fashion or prioritize them according to defined rules.
PYTHON
class PriorityQueue:
    def __init__(self, max_priority):
        self.queues = [[] for _ in range(max_priority + 1)]

    def enqueue(self, task, priority):
        if 0 <= priority < len(self.queues):
            self.queues[priority].append(task)

    def dequeue(self):
        # Process queues from highest to lowest priority
        for i in reversed(range(len(self.queues))):
            if self.queues[i]:
                return self.queues[i].pop(0)
Aging Mechanism

An aging mechanism can be implemented by periodically re-evaluating tasks and redistributing them based on their age or priority changes.

#### Implementation

Task Structure: Add an "age" field to each task.
Aging Function: Periodically run a function to re-evaluate tasks:
If a high-priority task has been in the queue for too long, it is moved to a lower priority queue.
Lower priority tasks that have been waiting longer can be bumped up if resources allow.
PYTHON
class Task:
    def __init__(self, id, priority):
        self.id = id
        self.priority = priority
        self.age = 0

def age_tasks(tasks, max_age, min_priority_increase):
    current_time = time.time()
    for task in tasks:
        if (current_time - task.last_seen) > max_age and task.priority < min_priority_increase:
            task.priority += min_priority_increase

#### Closing Summary
+ Common Mistakes/Follow-Ups

Using multiple queues provides clear separation but can be cumbersome to manage as the number of priorities increases. An aging mechanism is more dynamic, ensuring that tasks don't get stuck indefinitely, but it requires careful tuning to avoid over-aging and under-utilizing resources.

#### Common Mistakes

Ignoring Fairness: Focusing solely on processing high-priority tasks without mechanisms for low-priority task fairness.
Inefficient Aging: Over-aggressive aging can lead to unnecessary reprocessing of tasks, while too conservative aging might not resolve starvation issues effectively.

For follow-ups, one might be asked about performance optimizations, resource management strategies, or how this system integrates with other components of a larger application.

**Code:**

```

class PriorityQueue: def init(self, max_priority): self.queues = [[] for _ in range(max_priority + 1)]

CODE
Copy
def enqueue(self, task, priority):
    if 0 <= priority < len(self.queues):
        self.queues[priority].append(task)

def dequeue(self):
    # Process queues from highest to lowest priority
    for i in reversed(range(len(self.queues))):
        if self.queues[i]:
            return self.queues[i].pop(0)
CODE
Copy

```python
class Task:
    def __init__(self, id, priority):
        self.id = id
        self.priority = priority
        self.age = 0

def age_tasks(tasks, max_age, min_priority_increase):
    current_time = time.time()
    for task in tasks:
        if (current_time - task.last_seen) > max_age and task.priority < min_priority_increase:
            task.priority += min_priority_increase
```

**Flow:**

1. enqueue(task, priority)
2. Add to appropriate queue
3. dequeue()
4. Process highest priority tasksfirst
5. age_tasks(tasks, max_age,min_priority_increase)
6. Run periodically
7. Tasks with age > max_age andpriority < min_priority_increaseare aged up

