---
type: "practice"
difficulty: "Deep"
---

# Explain the trade-off between AZ-affinity routing (cheaper, lower latency) and forced cross-AZ spread (resilience). As principal, how do you get most of the cost/latency benefit while guaranteeing you don't concentrate a service into one AZ that can then take it fully down?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Cross-AZ Traffic Costs](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Cross-AZ_Traffic_Costs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In distributed systems, designing routing strategies between Availability Zones (AZs) involves balancing two critical trade-offs: cost and latency versus resilience. AZ-affinity routing keeps related services within the same AZ to minimize latency and cost, but it can lead to a single-point-of-failure if that zone goes down. Forced cross-AZ spread ensures high availability by distributing traffic across multiple zones, which is more resilient but comes with higher costs due to increased network latency.

To address this, as a DevOps Architect/Principal, we need to find a balance where we get most of the benefits while mitigating risks. The key idea here is to use a hybrid approach that leverages both strategies based on service requirements and criticality.

**Comparison:**

```
Service Type	Routing Strategy
Critical (e.g., DB)	Cross-AZ-spread
High-latency-tolerant	AZ-affinity
Non-critical	AZ-affinity
```

**Code:**

```python
def routing_decision(service_type):
    if service_type in ["high-latency-tolerant", "non-critical"]:
        return "AZ-affinity"
    else:
        return "cross-AZ-spread"
```

**Example input / output:**

```

Given a service type, the routing decision is as follows:

PLAINTEXT
Copy
Input: routing_decision("database")
Output: cross-AZ-spread

Input: routing_decision("web-frontend")
Output: AZ-affinity

Input: routing_decision("data-processing")
Output: AZ-affinity
```

**Data:**

```json
{
  "service_types": {
    "database": "critical",
    "web-frontend": "high-latency-tolerant",
    "data-processing": "non-critical"
  }
}
```

**Flow:**

1. Yes
2. No
3. Service Type
4. Is Service Critical?
5. Use Cross-AZ-spread
6. Use AZ-affinity

