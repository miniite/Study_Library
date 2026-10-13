---
type: "practice"
difficulty: "Expert"
---

# A single bad feature-flag flip caused a global incident because it propagated instantly to every service. Design a flag-delivery system that combines fast propagation with blast-radius control: staged rollout of flag changes, automatic rollback on SLO regression, and per-cell flag scoping.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Shared Config as Single Point of Failure](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Shared_Config_as_Single_Point_of_Failure.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview

In distributed systems, managing feature flags efficiently is crucial to minimize risk while enabling fast experimentation and deployment. The challenge here is designing a system that ensures rapid feature rollouts with controlled blast radius, automatic rollback mechanisms on service-level objectives (SLO) regressions, and per-cell scoping of flags.

#### Triaging the Question

The question requires us to design a robust flag-delivery system that addresses several key aspects:

Staged Rollout: Gradually deploy changes to avoid sudden impacts.
Automatic Rollback: Automatically revert changes if they degrade performance metrics.
Blast Radius Control: Limit the impact of rollouts to specific subsets of services or cells.

#### Assumptions
and Reasoning

#### Assumptions

We have multiple microservices forming a distributed system.
Each service has defined SLOs (e.g., response time, error rate).
The system needs to handle millions of requests per second with minimal downtime.
Services are partitioned into cells for easier management.
Step-by-Step Solution

Flag Delivery Mechanism:

Implement a centralized flag service that manages all feature flags and their state transitions.
Ensure the flag service is highly available (N+1 or N+2 redundancy) to handle sudden spikes in traffic.

Staged Rollout Strategy:

Divide services into cells based on geographical location, business logic, or any other relevant criteria.
Deploy changes incrementally by rolling out to one cell at a time and monitoring SLOs before moving to the next cell.
Use a canary release approach where a small percentage of users are exposed to the new feature.

SLO Monitoring:

Continuously monitor each service's SLOs using metrics like response time, error rate, etc.
Set up alerts and triggers that automatically initiate rollbacks if SLO thresholds are breached.

Rollback Mechanism:

Implement a rollback strategy that instantly reverts the feature flag to its previous state upon detection of an SLO regression.
Ensure fast failover capabilities by maintaining redundant components in the flag delivery system.

Per-Cell Scoping:

Define service cells and associate flags with specific cells or subsets of services.
Allow fine-grained control over which services receive a particular feature at any given time.
Full Solution
Centralized Flag Service

The central flag service will be responsible for:

Flag Store
Service Cell
Staged Rollout Controller
SLO Monitoring Alerts
Rollback Trigger
Feature Flag Reversion
Staged Rollout Controller

The staged rollout controller will manage the gradual deployment of feature flags:

Flag Store
Staged Rollout
Service Cell
User Exposure
Metrics Collection
SLO Check
Revert if SLO Breached
SLO Monitoring and Rollback

The system will monitor SLOs in real-time and trigger rollbacks as necessary:

Flag Store
Service Cell
Metrics Collection
SLO Check
Revert if SLO Breached

#### Closing Summary

In summary, the proposed solution leverages a centralized flag service for managed rollouts, staged deployments, and real-time monitoring of SLOs to ensure rapid but controlled feature delivery. The system's architecture is designed with redundancy, failover capabilities, and fine-grained control over service cells to minimize risk during deployments.

#### Common Mistakes/Follow-Ups
Lack of Redundancy: Failing to set up redundant components in the flag delivery system can lead to single points of failure.
Insufficient SLO Monitoring: Not having robust monitoring and alerting mechanisms can result in unnoticed regressions.
Overly Complex Rollbacks: Complicated rollback procedures can introduce delays or errors, leading to prolonged downtime.
Code Blocks
PYTHON
# Example of a simple flag state transition in Python
class Flag:
    def __init__(self, name):
        self.name = name
        self.state = "OFF"

    def set_state(self, new_state, cell):
        if self.slo_check(cell):  # Check SLOs before changing state
            self.state = new_state
            print(f"Feature '{self.name}' is now {new_state} in cell {cell}")

    def slo_check(self, cell):
        # Simulate SLO check logic
        return True

# Example usage
flag = Flag("New Feature")
flag.set_state("ON", "EastCell")  # Deploy to EastCell if SLOs are met

**Comparison:**

```
Cell	Services	SLO Metrics
East	serviceA, B	Response Time: <100ms, Error Rate: <0.01%
West	serviceC, D	Response Time: <200ms, Error Rate: <0.05%
```

**Code:**

```python
# Example of a simple flag state transition in Python
class Flag:
    def __init__(self, name):
        self.name = name
        self.state = "OFF"

    def set_state(self, new_state, cell):
        if self.slo_check(cell):  # Check SLOs before changing state
            self.state = new_state
            print(f"Feature '{self.name}' is now {new_state} in cell {cell}")

    def slo_check(self, cell):
        # Simulate SLO check logic
        return True

# Example usage
flag = Flag("New Feature")
flag.set_state("ON", "EastCell")  # Deploy to EastCell if SLOs are met
```

**Example input / output:**

```text
Input:
- Deploy new feature to EastCell

Output:
1. Flag state transition for "New Feature" to ON in EastCell
2. SLO check passes, no rollback triggered
3. Metrics collection and monitoring continue
```

**Data:**

```json
{
    "cell": {
        "EastCell": {
            "services": ["serviceA", "serviceB"],
            "slo_metrics": {"response_time_ms": 100, "error_rate": 0.01}
        },
        "WestCell": {
            "services": ["serviceC", "serviceD"],
            "slo_metrics": {"response_time_ms": 200, "error_rate": 0.05}
        }
    }
}
```

**Flow:**

1. Flag Store
2. Service Cell
3. Staged Rollout Controller
4. SLO Monitoring Alerts
5. Rollback Trigger
6. Feature Flag Reversion

