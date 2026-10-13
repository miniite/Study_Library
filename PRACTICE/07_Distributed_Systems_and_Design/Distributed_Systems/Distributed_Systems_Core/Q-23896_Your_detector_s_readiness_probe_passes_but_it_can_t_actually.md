---
type: "practice"
difficulty: "Intermediate"
---

# Your detector's readiness probe passes but it can't actually score because the model failed to load, so the LB routes telemetry to a zombie that silently drops detection. Explain deep versus shallow health checks here and the crash-loop risk of making liveness too deep.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Slow Dependencies and Load Shedding](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Deadline_Propagation/Slow_Dependencies_and_Load_Shedding.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this issue, we need to distinguish between "deep" and "shallow" health checks in a distributed system context, particularly focusing on Kubernetes readiness/liveness probes. The core of the problem is that the detector's readiness probe passes because it can initialize successfully, but its liveness check fails when trying to load the model, leading to the detector becoming a zombie pod that continues to receive traffic from the Load Balancer (LB). This scenario highlights the importance of correctly configuring these health checks.

#### Triage
and Assumptions
Readiness Probe: Ensures the container is up and ready to accept traffic.
Liveness Probe: Ensures the application inside the pod is healthy and running, preventing the pod from being terminated by the Kubernetes scheduler.
Model Loading Logic: The detector attempts to load a machine learning model during its initialization.
Deep vs. Shallow Health Checks
Shallow Health Check (Readiness): This check only ensures that the detector can initialize properly and is ready to start processing traffic. It doesn't verify internal application logic, like whether the model has been successfully loaded.
Deep Health Check (Liveness): This check verifies the overall health of the running application by executing a more extensive set of checks, such as ensuring the model is correctly loaded and accessible.
The Issue

In this scenario, the detector's readiness probe passes because it can initialize successfully. However, the liveness check fails when trying to load the model, causing the pod to be marked as unhealthy. Since the LB routes telemetry to a pod that has passed its readiness check but failed its liveness check, the detector becomes a zombie and continues to receive traffic silently dropping detection.

#### Solution
from First Principles
Separate Concerns: Clearly separate the readiness and liveness probes based on their responsibilities.
Readiness Probe:
Simple initialization logic: Ensure that necessary services are up, configurations are correct, etc.

#### Example
A simple HTTP GET request to a health endpoint to check if the detector is ready to start processing traffic.
YAML
readinessProbe:
  httpGet:
    path: /healthz
    port: 8080
Liveness Probe:
Comprehensive logic to ensure the application is fully functional, including model loading.

#### Example
A script or command that runs a suite of checks, such as validating model state, checking for any data anomalies, etc.
YAML
livenessProbe:
  exec:
    command:
      - /bin/bash
      - -c
      - "python3 -c 'import my_model; print(my_model.is_valid())'"
Zombie Pod Handling:
Ensure the LB and DNS updates are configured to handle zombie pods effectively.
Implement a strategy like rolling restarts or draining unhealthy nodes to prevent stale data.

#### Closing Summary

In summary, the issue arises from improperly configured health checks where the readiness probe only ensures basic initialization while the liveness probe performs comprehensive application validation. By properly separating and configuring these probes, we can ensure that only healthy detectors process telemetry traffic, mitigating the risk of zombie pods.

#### Common Mistakes/Follow-Ups
Mistake 1: Configuring both probes too deeply or shallowly, leading to false positives/negatives.
Mistake 2: Failing to implement effective handling for zombie pods, such as draining them from the load balancer.

#### Follow-Up
Question: How would you scale this solution in a highly available environment with multiple replicas? Consider stateful vs. stateless detectors and their implications on health check configurations.

**Code:**

```yaml
# Example readiness probe configuration
readinessProbe:
  httpGet:
    path: /healthz
    port: 8080

# Example liveness probe configuration
livenessProbe:
  exec:
    command:
      - /bin/bash
      - -c
      - "python3 -c 'import my_model; print(my_model.is_valid())'"
```

**Flow:**

1. FLOWCHART · MERMAID
2. graph TD
3. A[Readiness Check] --> B(Initialize Services)
4. B --> C{Is Detector Ready?}
5. C -- Yes --> D[Liveness Check]
6. C -- No --> E[Restart Pod]
7. D --> F(Liveness Check Passes) :: Model Valid
8. D --> G[Liveness Check Fails] :: Model Invalid
9. F --> H[Mark Pod Healthy]
10. G --> I[Determine Action: Draining, Restart, etc.]

