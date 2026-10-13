---
type: "practice"
difficulty: "Deep"
---

# A worker pool processes a priority queue but low-priority jobs starve indefinitely under sustained high-priority load; what aging or multi-level feedback structure guarantees eventual progress?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Bounded Blocking Queues](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Bounded_Blocking_Queues.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of low-priority jobs starving in a worker pool due to sustained high-priority loads, we need to design a mechanism that ensures both fairness and efficiency. The key idea here is to introduce an aging or multi-level feedback structure that progressively elevates the priority of low-priority jobs over time.

#### Triage
Understanding the Problem

The core issue is that under heavy load, low-priority jobs remain in a backlog indefinitely because high-priority jobs are always processed first. To ensure eventual progress for all jobs, we need to dynamically adjust their priorities based on how long they've been waiting.

#### First-Principles Solution

We can achieve this by implementing an aging system that increases the priority of low-priority jobs as time passes since their submission or last processing attempt. This aging mechanism should be designed in such a way that it guarantees that no job will starve indefinitely, while still maintaining reasonable performance and fairness among different priorities.

Key Components
Priority Queue: Maintain a primary priority queue where jobs are ordered based on their initial priority.
Aging Mechanism: Implement an aging function that adjusts the priority of low-priority jobs over time.
Feedback Loop: Periodically re-evaluate and adjust job priorities to ensure fairness.
Step-by-Step Solution
Initialization: Each job is initially assigned a priority based on its importance or urgency.
Aging Calculation: Define an aging function that increases the priority of low-priority jobs over time, e.g., by adding a fixed amount to their priority every second or per batch processed.
Priority Adjustment: Periodically, re-evaluate all jobs' priorities and update them in the queue.
Processing Order: Process jobs from the highest priority to the lowest, ensuring that aging jobs get an opportunity to be dequeued and executed.

#### Example

Let's consider a simplified example where we have two types of jobs: High (H) and Low (L). Initially, both are processed based on their priority:

Job H1 -> Priority 5
Job L1 -> Priority 3

After a certain period, the aging function increases the priority of low-priority jobs:

After 10 seconds: Job H2 -> Priority 6, Job L1 -> Priority 4 (L1's age increases its effective priority).

The updated queue would be:

H2 (Priority 6)
L1 (Priority 4)
H1 (Priority 5)

Processing order would then be:

H2
L1
H1

#### Closing Summary
and Common Mistakes/Follow-Ups

By implementing an aging mechanism, we ensure that low-priority jobs eventually get a chance to execute even under sustained high-priority load. However, the choice of aging function is critical; if it's too aggressive, it could introduce unnecessary overhead or skew the overall job processing.

#### Common Mistakes

Failing to define an appropriate aging function.
Overlooking the need for periodic re-evaluation and adjustment of priorities.
Ignoring the potential impact on real-time performance during aging updates.

#### Follow-Ups

How would you implement this in a distributed environment?
What are the trade-offs between different aging functions?
How do you ensure that the system remains efficient under heavy load?

**Comparison:**

```
Time	Jobs Priority
0	[5, 3]
5	[6, 4]
10	[7, 5]
```

**Code:**

```python
class Job:
    def __init__(self, priority):
        self.priority = priority
        self.time_since_submission = 0

def process_jobs(jobs, aging_factor=1, age_interval=5):
    while jobs:
        # Sort jobs based on current priorities
        jobs.sort(key=lambda x: (x.priority - x.time_since_submission * aging_factor), reverse=True)

        # Process the top job
        if jobs[0].priority > 0:
            print(f"Processing {jobs[0]}")
            del jobs[0]
        else:
            break

        # Age all jobs by one unit for each interval
        for job in jobs:
            job.time_since_submission += aging_factor

# Example usage
jobs = [Job(5), Job(3)]
process_jobs(jobs)
```

**Example input / output:**

```text
Initial jobs:
- Job1 (Priority: 5)
- Job2 (Priority: 3)

After 10 seconds of aging with an aging factor of 1:
- Job2's priority is increased to 4.

Processed order after aging and processing:
1. Job1
2. Job2
```

**Data:**

```json
{
  "job1": { "priority": 5, "time_since_submission": 0 },
  "job2": { "priority": 3, "time_since_submission": 0 }
}
```

**Flow:**

1. Yes
2. No
3. Initial Job Submission
4. Is Queue Empty?
5. Return
6. (Re-)Sort Jobs by Priority +Age
7. Process Top Job
8. Age All Jobs

