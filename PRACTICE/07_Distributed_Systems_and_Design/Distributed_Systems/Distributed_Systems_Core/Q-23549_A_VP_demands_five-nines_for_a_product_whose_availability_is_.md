---
type: "practice"
difficulty: "Expert"
---

# A VP demands five-nines for a product whose availability is structurally capped by its weakest hard dependency. Show how you'd compute the achievable ceiling from the dependency graph, communicate the honest number, and propose the isolation/degradation changes that would actually raise it.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Static Stability](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Static_Stability.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this challenge, we need to understand the availability of a system in terms of its weakest link. Given the VP's requirement for five-nines (99.999% availability), and knowing that the product is structurally limited by its weakest hard dependency, our task is to first determine the maximum achievable availability from the current dependency graph. We then explore potential changes—such as isolating or degrading certain services—that could improve this ceiling.

#### Triage

The question probes several aspects: understanding the availability of a system, identifying critical dependencies, and proposing practical solutions to enhance overall reliability. Our assumptions are that we have visibility into the system's dependency graph, including the current availability rates of each component, and that degrading or isolating components may be possible without impacting core functionality.

#### First-Principles Solution

Model the System:

Represent the system as a directed acyclic graph (DAG) where nodes are services/components, and edges represent dependencies.

Compute Current Availability:

For each service ( S ), let its availability be denoted by ( A(S) ).
The availability of any dependent service ( D ) that depends on ( S ) is calculated as ( A(D) = 1 - (1 - A(S)) \times p ), where ( p ) is the probability that ( S ) is available when needed.

Identify the Critical Path:

Traverse the dependency graph to find the path with the lowest availability, which represents the system's weakest link.

Propose Enhancements:

Analyze potential changes (isolation, degradation) and their impact on overall availability.

#### Example

Let’s assume we have a simplified dependency graph:

ServiceA
ServiceB
ServiceC

Given the following availability rates:

( A(ServiceA) = 0.995 )
( A(ServiceB) = 0.98 )
( A(ServiceC) = 0.97 )

Compute Availability for Service B:

( A(ServiceB) = 1 - (1 - 0.995) \times 0.02 = 0.99989 \approx 0.9999 )

Compute Availability for Service C:

( A(ServiceC) = 1 - (1 - 0.9999) \times 0.07 = 0.99994 )

The critical path here is from ( ServiceA ) to ( ServiceB ), with an availability of approximately ( 0.9999 ).

Raising the Ceiling

To raise this ceiling, we need to focus on improving or isolating ( ServiceA ):

Isolation:

Identify non-essential services dependent on ( ServiceA ) and isolate them if possible.

Degradation:

Implement strategies like circuit breakers or fail-open/fail-closed mechanisms in ( ServiceB ) to handle potential failures upstream.
Practical Steps

Analyze Dependencies:

Use tools like Istio or Linkerd to analyze and visualize dependencies.

Metrics Collection:

Ensure robust monitoring with Prometheus, Grafana, etc., to continuously track availability metrics.

A/B Testing:

Gradually implement changes in a controlled environment before rolling out fully.

#### Closing Summary

By understanding the dependency graph and its impact on overall system availability, we can identify the weakest links and propose practical solutions such as isolation or degradation to improve reliability. This approach ensures that resources are focused where they provide the most significant impact on the critical path of the system.

#### Common Mistakes/Follow-Ups
Ignoring Non-Critical Paths: Failing to recognize and isolate non-critical dependencies.
Over-Estimating Impact: Assuming changes will have a more significant impact than they actually do.
Neglecting Monitoring: Not having robust monitoring in place can lead to delays in identifying and addressing issues.

**Comparison:**

```
Service	Availability	Probability of Failure	Dependency Of
A	0.995	-	B
B	0.98	0.02	A
C	0.97	0.07	B
```

**Code:**

```python
# Example Python code for availability calculation
def calculate_availability(current_service_availability, dependency_probability):
    return 1 - (1 - current_service_availability) * dependency_probability

availability_a = 0.995
dependency_b_on_a = 0.02
availability_b = calculate_availability(availability_a, dependency_b_on_a)

# Further calculations for ServiceC and the critical path
```

**Example input / output:**

```text
Input:
- ServiceA availability: 0.995
- ServiceB availability: 0.98 (depends on ServiceA with probability 0.02)
- ServiceC availability: 0.97 (depends on ServiceB with probability 0.07)

Output:
- Availability of ServiceB: 0.99989
- Critical path: ServiceA -> ServiceB, total availability: ~0.9999
```

**Data:**

```json
{
    "services": {
        "ServiceA": {"availability": 0.995},
        "ServiceB": {"availability": 0.98, "dependency_of": ["ServiceA"], "probability_of_failure": 0.02},
        "ServiceC": {"availability": 0.97, "dependency_of": ["ServiceB"], "probability_of_failure": 0.07}
    }
}
```

**Flow:**

1. Services
2. ServiceA
3. ServiceB
4. ServiceC
5. ServiceA
6. ServiceB
7. ServiceC

