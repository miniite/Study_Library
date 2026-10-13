---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd set the org's policy on running stateful workloads (databases, brokers) on Kubernetes versus managed services versus dedicated infra, and how operational maturity factors into the call.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Stateless by Default](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Stateless_by_Default/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we're tasked with determining the optimal placement of stateful workloads like databases and message brokers within a Kubernetes cluster. The decision hinges on balancing cost, performance, reliability, and operational maturity. By understanding where to host these critical applications, we can leverage the strengths of each option—Kubernetes clusters, managed services, or dedicated infrastructure—to achieve our objectives.

To set the organization's policy effectively, we need to consider several factors:

Operational Maturity: The level of expertise within the team.
Cost Considerations: Balancing upfront costs and ongoing maintenance.
Performance Requirements: Meeting SLAs for latency and availability.
Scalability Needs: Ensuring the solution can handle growth.
Security Policies: Aligning with organizational security standards.

The core idea is to align our policy choices with these factors, ensuring a balanced approach that maximizes benefits while minimizing risks. We'll start by outlining the pros and cons of each option before making a strategic decision.

**Comparison:**

| Placement Option       | Pros                                                                                           | Cons                                                                                          |
|------------------------|------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------|
| Managed Services       | Lower operational overhead; easier scaling and management                                     | Limited customization; higher costs for advanced features                                   |
| Kubernetes Cluster     | Full control over resources; better performance tuning                                          | Higher initial investment; requires expertise in ops; more complex setup                    |
| Dedicated Infrastructure | Easier to secure; proven reliability with established hardware                               | High upfront cost; less flexible than managed services                                       |

**Code:**

```
MERMAID
Copy
graph LR;
    A[Stateful Workload] --> B1{Is the team mature enough?}
    B1 -->|Yes| C1[Migrate to Managed Services]
    B1 -->|No| D1[Dedicated Infrastructure]
    C1 --> E1[Review Performance and Cost]
    D1 --> F1[Kubernetes Cluster - Self-Managed]
    E1 --> G1[Evaluate Scalability Needs]
    G1 --> H1[Operational Maturity Check]
    H1 --> I1[Finalize Policy]
```

**Example input / output:**

```text
| Team Maturity Level | Initial Investment (Kubernetes) | Maintenance Costs (Managed Services) |
|---------------------|---------------------------------|--------------------------------------|
| Low                 | High                            | Medium                               |
| Moderate            | Medium                          | Low                                  |
| High                | Low                             | Low                                  |

Given a moderate team maturity, the initial investment for Kubernetes is medium, while maintenance costs are low for managed services. This suggests that moving to managed services could be more cost-effective.
```

**Data:**

```json
{
  "workloadType": "stateful",
  "placementOptions": {
    "managedServices": ["Database", "Message Broker"],
    "kubernetesCluster": ["Custom Applications", "High-Performance Workloads"],
    "dedicatedInfrastructure": ["Legacy Systems", "Low-Maintenance"]
  },
  "teamMaturityLevel": [1, 2, 3],
  "costFactors": {
    "initialInvestment": "High",
    "maintenanceCosts": "Medium",
    "operationalCosts": "Low"
  }
}
```

**Flow:**

1. Yes
2. No
3. Stateful Workload
4. Is the team mature enough?
5. Migrate to Managed Services
6. Dedicated Infrastructure
7. Review Performance and Cost
8. Kubernetes Cluster -Self-Managed
9. Evaluate Scalability Needs
10. Operational Maturity Check
11. Finalize Policy

