---
type: "practice"
difficulty: "Expert"
---

# Incident retro at scale: a merge queue you introduced improved main stability but during a traffic-heavy launch it serialized deploys and delayed a critical fix by 40 minutes. How do you redesign the queue to preserve stability while allowing priority preemption for urgent changes?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Standard Deployment Strategies](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Rollback_Strategy/Standard_Deployment_Strategies.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we need to redesign our merge queue to ensure that it maintains its effectiveness in stabilizing deployments during regular operations but also allows critical updates to be deployed urgently without compromising overall stability. The key challenge is balancing throughput with the ability to prioritize high-priority changes.

#### Triage

The core question here involves understanding how to manage a merge queue in such a way that:

Stability: We need to ensure that non-critical deployments do not significantly impact the system's performance and availability.
Priority Preemption: In cases where urgent updates are required, we must allow these changes to be deployed ahead of regular merges without causing undue risk.

#### Assumptions
The merge queue currently handles a large number of deployments.
There is an existing mechanism for prioritizing critical issues (e.g., tickets or alerts).
Our system has the capability to deploy changes in multiple stages, such as rolling out updates gradually over time.

#### First Principles Solution

To address this issue, we can redesign our merge queue by introducing a multi-tier deployment strategy that allows us to handle both regular and urgent deployments effectively. This involves:

Priority Levels: Assigning different priority levels to changes.
Queuing Mechanism: Implementing a queuing mechanism that respects these priorities but still maintains overall stability.
Step-by-Step Solution
1. Define Priority Levels

We can define three priority levels: Critical, Routine, and Scheduled.

Critical (highest): For urgent updates that require immediate deployment.
Routine: Standard deployments that follow the regular queue.
Scheduled: Deployments that are planned ahead of time.
2. Implement a Multi-Tier Queue

We can use a prioritized queue where:

High-priority Critical changes get pushed to a separate, high-priority queue.
The main merge queue processes lower priority Routine and Scheduled changes in a FIFO manner.

Here’s how the system might look:

Process
No
Yes
Queue
Is change critical?
Scheduled Queue
Critical Preemptive Queue
Deploy at scheduled time
Deploy immediately
3. Deploy Mechanism

For the Critical queue, we can deploy changes as soon as they are ready without waiting for other deployments to complete. For the main merge queue, we continue to use a stable deployment strategy that ensures no single change significantly impacts the system.

#### Closing Summary
+ Common Mistakes/Follow-Ups

By implementing a multi-tier queue and prioritizing critical updates, we can ensure that urgent changes do not delay regular deployments while still maintaining overall stability. The key is to balance urgency with the need for a stable deployment process. Some common mistakes to avoid include:

Ignoring the importance of stability in favor of too many preemptions.
Overloading the system by pushing too many high-priority changes at once.

For follow-ups, we would need to consider factors like how often critical issues occur, the complexity of deploying urgent changes, and potential fallback strategies if a critical update fails.

**Comparison:**

```
Priority	Description
Critical	High urgency, deploy immediately
Routine	Standard deployment order
Scheduled	Deploy at a planned time
```

**Code:**

```python
# Example code for queue management
class MergeQueue:
    def __init__(self):
        self.critical_queue = []
        self.routine_queue = []

    def add_change(self, change):
        if change.priority == 'Critical':
            self.critical_queue.append(change)
        else:
            self.routine_queue.append(change)

    def process_changes(self):
        while self.critical_queue:
            # Process high-priority changes immediately
            critical_change = self.critical_queue.pop(0)
            deploy_critical_change(critical_change)

        while self.routine_queue:
            # Process routine changes in a stable manner
            routine_change = self.routine_queue.pop(0)
            deploy_routine_change(routine_change)
```

**Example input / output:**

```text
Input:
- Add a new critical change with ID=1 and priority 'Critical'
- Add routine changes with IDs 3 and 4

Output:
- Critical change with ID=1 is deployed immediately
- Routine changes are processed in the routine queue
```

**Data:**

```json
{
  "queue": {
    "critical": [
      {"id": 1, "priority": "Critical", "type": "Bug Fix"},
      {"id": 2, "priority": "Critical", "type": "Security Patch"}
    ],
    "routine": [
      {"id": 3, "priority": "Routine", "type": "Feature Update"},
      {"id": 4, "priority": "Scheduled", "type": "Performance Optimization"}
    ]
  }
}
```

**Flow:**

1. No
2. Yes
3. Queue
4. Is change critical?
5. Scheduled Queue
6. Critical Preemptive Queue
7. Deploy at scheduled time
8. Deploy immediately

