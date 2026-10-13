---
type: "practice"
difficulty: "Deep"
---

# You detect and want to auto-scale a service on queue depth, but the autoscaler and a downstream failing dependency created a death spiral where scaling out made recovery impossible. Reconstruct the feedback loop and the guardrails (cooldown, dependency-aware scaling) you'd add.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Metastable Failures](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Retry_Budgets_and_Backoff/Metastable_Failures.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of auto-scaling in a distributed system where an autoscaler's response to queue depth leads to a death spiral with downstream dependencies, we need to design a robust feedback loop and implement guardrails like cooldown periods and dependency-aware scaling policies. The key is to ensure that the autoscaler doesn't exacerbate the situation by making recovery harder.

Triaging the Problem

The core issue here involves a service's auto-scaling logic responding to increased queue depth, which in turn causes more instances to be spun up. However, if these new instances fail due to a downstream dependency (e.g., database or another microservice), this triggers further scaling, leading to an uncontrollable feedback loop that can bring the system down.

Reconstructing the Feedback Loop
Queue Depth Monitoring: The auto-scaler should continuously monitor the queue depth and adjust the number of instances based on predefined thresholds.
Cooldown Periods: Introduce a cooldown period after scaling actions, during which no additional scaling is performed to allow time for recovery.
Dependency-Aware Scaling: Ensure that the autoscaler takes into account the health status of downstream dependencies before making scaling decisions.

#### First-Principles Solution

Detection and Response:

When the queue depth exceeds a certain threshold, scale out by increasing the number of service instances.
Implement a cooldown period to prevent immediate re-scaling if the issue is transient or being resolved.

Dependency Health Check:

Periodically check the health of downstream dependencies before allowing scaling actions.
If any critical dependency fails, delay or skip scaling until the dependency recovers.

Feedback Loop Management:

Use a state machine to manage the scaling states: idle, scaling in progress, cooldown, and failed recovery.
Transition between these states based on queue depth changes and health checks of dependencies.

#### Closing Summary
+ Common Mistakes/Follow-Ups

By implementing a robust feedback loop with cooldown periods and dependency-aware scaling policies, we can mitigate the risk of creating an unmanageable death spiral. Common mistakes include not having sufficient cooldown periods, failing to factor in dependency health during scaling decisions, or neglecting to handle edge cases such as transient failures.

To further solidify this solution:

Edge Cases: Consider what happens if all instances fail due to a critical dependency.
Performance Impact: Evaluate how the additional checks for dependencies might impact overall system performance and latency.
Scalability: Ensure that the feedback loop can handle scaling events across multiple services with varying dependencies.

**Comparison:**

```
State	Action	Condition
Idle	Monitor queue depth	Queue depth > threshold
Scaling In Progress	Wait for cooldown	Time since last scale < cooldown_period
Cooldown	Delay scaling action	Dependency is unhealthy
Failed Recovery	Skip scaling	All dependencies are down, retry after interval
```

**Code:**

```python
# Example Python pseudo-code for autoscaler logic

class AutoScaler:
    def __init__(self, queue_depth_threshold, cooldown_period, dependency_health_check_interval):
        self.queue_depth_threshold = queue_depth_threshold
        self.cooldown_period = cooldown_period
        self.dependency_health_check_interval = dependency_health_check_interval
        self.instances = 1
        self.last_scaling_time = time.time()
        self.dependency_status = {}

    def scale_out(self, new_instances):
        if (time.time() - self.last_scaling_time) < self.cooldown_period:
            return

        # Check dependencies health
        for dependency in self.dependencies:
            status = check_dependency_health(dependency)
            if not status:
                print("Dependency is unhealthy. Delaying scaling.")
                return

        # Scale out
        self.instances += new_instances
        self.last_scaling_time = time.time()
        print(f"Scaling out to {self.instances} instances.")

    def scale_in(self, old_instances):
        self.instances -= old_instances
        print(f"Scaling in to {self.instances} instances.")
```

**Example input / output:**

```text
Input:
- Queue depth exceeds threshold.
- Dependency health check indicates a critical service is down.

Output:
- Autoscaler delays scaling action for cooldown period (300 seconds).
- No instances are scaled out until the dependency recovers.
```

**Data:**

```json
{
  "queue_depth_threshold": 100,
  "cooldown_period": 300, // 5 minutes
  "dependency_health_check_interval": 60 // 1 minute
}
```

**Flow:**

1. Yes
2. Healthy
3. Unhealthy
4. Idle
5. Monitor Queue Depth
6. Queue Depth > Threshold?
7. Scale Out Instances
8. Cooldown Period
9. Check Dependency Health
10. Wait for Cooldown Period
11. Scale Out Instances
12. Delay Scaling Action
13. Fallback to Idle State

