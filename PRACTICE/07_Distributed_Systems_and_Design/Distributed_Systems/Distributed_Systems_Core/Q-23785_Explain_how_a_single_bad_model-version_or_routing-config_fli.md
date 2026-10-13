---
type: "practice"
difficulty: "Expert"
---

# Explain how a single bad model-version or routing-config flip could propagate instantly to your entire inference fleet and cause a global incident. Design a staged, blast-radius-scoped rollout for model and config changes with automatic SLO-and-quality-based rollback, and why the flag system itself must not be a hot-path SPOF.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Shared Config as Single Point of Failure](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Shared_Config_as_Single_Point_of_Failure.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In distributed systems, a single bad model version or routing configuration can indeed propagate rapidly, leading to a global incident. As an ML engineer with expertise in distributed systems, I'll explain how such incidents occur and propose a staged rollout strategy that includes automatic rollback capabilities.

#### Triaging the Question

The question is probing our understanding of deployment processes, resilience strategies, and fault tolerance mechanisms in large-scale inference environments. The key aspects include:

Propagation Mechanism: How can a single bad model version or routing config affect an entire fleet?
Staged Rollout: How to introduce changes gradually to avoid a global impact.
Automatic Rollback: Ensuring that quality and service level objectives (SLOs) are monitored, and automatic rollback triggers if issues arise.
Non-Hot-Path Flag System: Why the flag system itself should not be a single point of failure.

#### First Principles Solution
Propagation Mechanism

In a distributed system, model versions or routing configurations can propagate through multiple layers:

Model Serving Layer: Models are served via APIs.
Routing Layer: Traffic is routed to different models based on the configuration.
Client Applications: These applications use the API to make predictions.

A single bad version in the model serving layer or a misconfiguration in the routing layer can cause all downstream services to fail, leading to a cascading effect across the entire fleet.

Staged Rollout

To mitigate this risk, we need to implement a staged rollout strategy that:

Gate Traffic Gradually: Introduce changes gradually and monitor their impact.
Use Canary Releases: Release new versions or configurations to a small subset of users first.
Monitor SLOs: Continuously check if the new version meets the desired performance metrics.
Automatic Rollback

To ensure high availability, we should:

Define Quality Metrics: Establish clear success criteria for both model and routing changes.
Automate Monitoring: Use automated monitoring tools to track these metrics in real-time.
Implement Rollback Mechanism: If the new version fails to meet SLOs, automatically revert to the previous stable state.
Non-Hot-Path Flag System

The flag system itself must be designed as a non-hot-path component to prevent it from becoming a single point of failure (SPOF):

Decouple Flags from Critical Paths: Ensure flags are not part of the main execution path.
Use Fail-Safe Mechanisms: Implement fail-safe logic that ensures the system can degrade gracefully if the flag system fails.

#### Closing Summary
and Common Mistakes

In summary, a well-architected distributed system should have mechanisms in place to prevent single points of failure, monitor quality metrics in real-time, and automatically rollback changes when they do not meet expected performance. The flag system itself must be designed as a non-critical component to avoid introducing additional SPOFs.

#### Common mistakes
include:

Failing to Monitor Quality Metrics: Not having continuous monitoring can lead to undetected failures.
Not Implementing Rollback Mechanisms: Without automatic rollback, issues may persist indefinitely.
Using the Flag System as a Hot Path: This can introduce additional SPOFs and complicate fault tolerance.
Final Thoughts

By following these principles, we can ensure that our distributed inference system is robust, resilient, and capable of handling changes with minimal risk to overall service quality.

**Comparison:**

```
Stage	Model Version	Routing Config	Duration
1	v1.0	routing-v2	24h
2	v1.1	routing-v3	72h
```

**Code:**

```python
# Example code for monitoring and logging
def monitor_model_slo(model_version):
    # Check if the model version meets SLOs
    if not check_slo(model_version):
        log.error("Model version failed SLOs, triggering rollback.")
        rollback_to_stable_version()
```

**Example input / output:**

```text
# Example input and output for monitoring metrics
Input: model_version=v1.0, metric=accuracy, target=95%
Output: accuracy=94.8%, result=FAIL, triggering rollback to v0.9
```

**Data:**

```yaml
# Example configuration for staged rollout
staging:
  - version: v1.0
    config: routing-v2
    duration: 24h
  - version: v1.1
    config: routing-v3
    duration: 72h
```

**Flow:**

1. Yes
2. No
3. Yes
4. Model Serving Layer
5. Is New Version?
6. Canary Release
7. Monitor Metrics
8. Meet SLOs?
9. Rollback to Previous Version
10. Gradually Gate Traffic

