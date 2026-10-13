---
type: "practice"
difficulty: "Deep"
---

# How do you architect cost governance for a feature-flag / experimentation platform where every running experiment duplicates infra paths, and stale experiments silently accrue cost forever? Design the experiment-lifecycle cost hygiene.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Shadow Mode for Cost Policies](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Shadow_Mode_for_Cost_Policies.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Cost governance in a feature-flag/experimentation platform can be complex due to the dynamic nature of experiments and their associated infrastructure. In this scenario, where every running experiment duplicates infra paths and stale experiments silently accrue cost forever, we need to design a robust solution that ensures efficient use of resources while preventing unnecessary costs. The key idea is to integrate automatic lifecycle management for experiments with cost monitoring and alerting.

To start, let's break down the problem into its core components: understanding how experiments are created and managed, implementing a cost-effective infrastructure setup, and ensuring timely termination or de-provisioning of unused experiments.

**Comparison:**

| Experiment ID | Start Time            | Status       | Resources                   |
|---------------|-----------------------|--------------|----------------------------|
| exp-001       | 2023-10-01T09:00:00Z  | running      | db-cluster, cache-service    |
| exp-002       | 2023-10-05T14:00:00Z  | completed    |                             |
| exp-003       | 2023-10-07T10:00:00Z  | marked for del| cache-service               |

**Code:**

```python
# Example code snippet for lifecycle management in an experiment platform
class Experiment:
    def __init__(self, id, start_time):
        self.id = id
        self.start_time = start_time

    def mark_for_deletion(self):
        # Mark the experiment to be deleted after a grace period
        pass

    def terminate(self):
        # Terminate the experiment and its associated resources
        pass
```

**Example input / output:**

| Input | Output |
|-------|--------|
| Create Experiment exp-003 on 2023-10-07T10:00:00Z with resources ["cache-service"] | New experiment marked for deletion after 30 days |
| Exp-001 completed on 2023-10-05T15:00:00Z | Resources associated with exp-001 are de-provisioned |

**Data:**

```json
{
  "experiments": [
    {
      "id": "exp-001",
      "start_time": "2023-10-01T09:00:00Z",
      "status": "running",
      "resources": ["db-cluster", "cache-service"]
    },
    {
      "id": "exp-002",
      "start_time": "2023-10-05T14:00:00Z",
      "status": "completed",
      "resources": []
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Experiment Created
6. Is Experiment Marked forDeletion?
7. De-provision Resources
8. Check if Expiration DatePassed
9. Renew Resources and ExtendGrace Period

