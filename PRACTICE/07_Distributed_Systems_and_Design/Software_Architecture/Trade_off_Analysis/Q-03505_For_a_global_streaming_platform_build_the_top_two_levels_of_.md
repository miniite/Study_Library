---
type: "practice"
difficulty: "Deep"
---

# For a global streaming platform, build the top two levels of a utility tree (quality attributes and their refinements) and justify why you ordered availability and performance the way you did given the business.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Utility Trees](<../../../../07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Utility_Trees.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To tackle this Architectural Trade-off Analysis (ATAM) question, we'll focus on building a utility tree for a global streaming platform to identify the top two levels of quality attributes and their refinements. We will prioritize availability over performance based on business requirements and justify our choice. Let's break down the process step-by-step.

#### Triaging the Question

The question requires us to construct a utility tree with at least two levels (quality attributes and their refinements) for a global streaming platform. The key quality attribute we'll focus on is availability, followed by performance. We need to justify our ordering of these attributes based on business priorities and technical considerations.

Step-by-First Principles Solution

Identify the Quality Attributes:

Availability: Ensuring the system remains operational under various conditions.
Performance: Ensuring the system operates efficiently, with low latency and high throughput.

Refine Availability:

Scalability: The ability to handle increased load without significant impact on performance.
Fault Tolerance: The resilience of the system against failures in individual components.
Redundancy: The presence of multiple backup systems or resources to ensure continuous service.

Refine Performance:

Latency: Time taken for a request to be processed and response returned.
Throughput: Number of requests processed per unit time.
Resource Utilization: Efficient use of CPU, memory, network bandwidth, etc.
Justifying the Order
Business Context:

For a global streaming platform, availability is often prioritized over performance due to several reasons:

User Experience:
Users expect uninterrupted viewing experience; any interruption can lead to user dissatisfaction and potential loss.
Reputation Management:
Frequent outages can damage the company’s reputation and deter potential users from subscribing or recommending the service.
Technical Considerations:
Fault Tolerance and Redundancy:
Implementing a redundant architecture ensures that even if one server fails, another can take over seamlessly.
Scalability:
Ensuring the system can scale horizontally to handle increases in traffic is crucial for maintaining availability during peak times.
Detailed Utility Tree
Availability
Scalability
Fault Tolerance
Redundancy
Auto-scaling
Autoscaling Groups
Replication
Failover Mechanisms
Distributed Clusters
Geographic Redundancy

#### Closing Summary

In summary, for a global streaming platform, we prioritized availability over performance. This decision is driven by the critical importance of uninterrupted service to user experience and business reputation. The utility tree shows how availability can be further refined into scalability, fault tolerance, and redundancy, with each attribute contributing to the overall resilience and reliability of the system.

#### Common Mistakes/Follow-Ups
Misordering Attributes: Ensuring that the quality attributes are ordered based on their criticality to the business is essential. Misordering can lead to suboptimal solutions.
Ignoring Trade-offs: Failing to recognize trade-offs between different attributes, such as the cost of implementing high availability versus improved performance.
Overlooked Edge Cases: Not considering edge cases like network outages or extreme load scenarios that could impact both availability and performance.

**Comparison:**

```
Quality Attribute	Refinement 1	Refinement 2
Availability	Scalability	Fault Tolerance
Performance	Latency	Throughput
```

**Code:**

```python
# Example code for auto-scaling implementation
def update_autoscaling_group(group_name, new_instances):
    # Code to update the autoscaling group with new instances
    pass

update_autoscaling_group("streaming-platform", 10)
```

**Example input / output:**

```text
Input:
- User requests a streaming session from the platform.
Output:
- Platform checks for available resources and starts streaming immediately, with minimal latency.
```

**Data:**

```json
{
  "availability": {
    "scability": ["auto-scaling", "autoscaling_groups"],
    "fault_tolerance": ["replication", "failover_mechanisms"],
    "redundancy": ["distributed_clusters", "geographic_redundancy"]
  },
  "performance": {
    "latency": [],
    "throughput": [],
    "resource_utilization": []
  }
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Check User Request
5. Available Resources?
6. Start Streaming Session
7. Auto-Scaling Triggered
8. Resources Allocated
9. Streaming Session Started

