---
type: "practice"
difficulty: "Deep"
---

# Your distributed training loss suddenly diverges to NaN on a multi-node run but was fine on a single node. What distributed-specific causes would you suspect first, and how do you isolate a bad rank or a gradient-sync bug?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Distributed Training Clusters](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Distributed_Training_Clusters.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with a sudden divergence to NaN in a distributed training setup after it was working fine on a single node, there are several key distributed-specific causes we should investigate first. These include issues related to gradient synchronization, communication delays, and potential bugs in the distributed environment setup.

To systematically identify these problems, let's start by examining the fundamental assumptions about our system and then proceed step-by-step through likely failure modes.

**Comparison:**

```
Initial System Setup
Isolate a specific rank?
Check for GradientSynchronization Issues
Inspect Communication Delays
Reproduce the Issue withSmaller Groups
Review DistributedEnvironment Configuration
```

**Example input / output:**

| Rank | Status |
|------|--------|
| 0    | Running |
| 1    | Running |
| ...  |         |
| 6    | Running |
| 7    | NaN     |

**Data:**

```json
{
  "distributed_setup": {
    "nodes": 8,
    "backend": "torch.distributed",
    "loss_function": "MSE"
  },
  "training_status": [
    {"rank": 0, "status": "running"},
    {"rank": 1, "status": "running"},
    ...
    {"rank": 7, "status": "NaN"}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Initial System Setup
4. Isolate a specific rank?
5. Check for GradientSynchronization Issues
6. Inspect Communication Delays
7. Reproduce the Issue withSmaller Groups
8. Review DistributedEnvironment Configuration

