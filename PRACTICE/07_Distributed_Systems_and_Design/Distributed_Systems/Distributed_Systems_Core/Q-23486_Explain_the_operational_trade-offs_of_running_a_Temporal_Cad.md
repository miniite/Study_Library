---
type: "practice"
difficulty: "Deep"
---

# Explain the operational trade-offs of running a Temporal/Cadence cluster yourself versus a managed offering for durable workflows. What failure modes of the persistence store and matching service do you have to operate?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Choosing an Event Backbone](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Choosing_an_Event_Backbone.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deciding between running a Temporal or Cadence cluster yourself versus using a managed offering, we need to consider several operational trade-offs, including cost, complexity, reliability, and support. This decision impacts not only the initial setup but also ongoing operations and maintenance of durable workflows.

#### Triaging the Question

The question revolves around understanding the operational differences between self-hosting a Temporal or Cadence cluster versus using a managed service. The key aspects being probed are:

Cost: Including hardware, software licensing, and human resources.
Complexity: Setting up, maintaining, scaling, and securing the infrastructure.
Reliability: Failure modes of the persistence store and matching service.
Support: Availability of managed services' support versus self-maintained systems.

#### Assumptions
We are comparing a managed offering with a self-hosted cluster on cloud or on-premises infrastructure.
The focus is on durability and reliability of workflows, as these systems handle critical business logic.
Both Temporal and Cadence offer robust solutions for managing durable workflows.

#### First Principles Solution
Self-Hosting

Advantages:

Complete Control: Full control over the stack, including custom configurations and optimizations.
Tight Integration: Easier to integrate with existing infrastructure and tools.
Performance Tuning: Precise tuning of resources and performance.

Disadvantages:

Cost: Non-trivial costs for hardware, software licenses (if any), and human expertise.
Complexity: Requires significant operational effort for setup, maintenance, and scaling.
Security: Increased responsibility for securing the infrastructure against threats like DDoS attacks or data breaches.
Maintenance: Continuous monitoring, updates, and troubleshooting are required.
Managed Offering

Advantages:

Cost-Effective: Lower upfront costs due to no need for hardware procurement and maintenance.
Ease of Use: Simplified setup and management, with minimal operational overhead.
Automatic Scaling: The provider handles scaling based on demand.
Support: Access to professional support teams for issues and escalations.

Disadvantages:

Lack of Customization: Limited flexibility in configuration compared to self-hosted solutions.
Dependency on Provider: Reliance on the provider’s service levels, availability, and updates.
Cost Fluctuations: Potential for unexpected cost increases due to usage-based billing.
Failure Modes

For both systems, we need to consider the failure modes of the persistence store (where workflow data is stored) and the matching service (responsible for task assignment).

Persistence Store Failure Modes:
Data Loss: Corruption or loss of data in the storage layer can lead to failed workflows.
Latency Issues: High latency or unavailability during reads/writes can impact workflow execution.
Replication Delays: Inconsistencies between primary and secondary replicas can cause delays in task completion.
Matching Service Failure Modes:
Task Assignment Failures: Misrouting of tasks due to service failure can lead to incorrect workflows.
Resource Overload: Excessive load on the matching service can result in bottlenecks and timeouts.
Synchronization Issues: Data synchronization delays between nodes can cause inconsistencies.

#### Conclusion

Choosing between self-hosting a Temporal or Cadence cluster and using a managed offering depends on the specific needs of your organization, budget constraints, and operational expertise. A managed offering offers lower initial costs, ease of use, and built-in support but comes with limitations in customization and dependency on the provider. Self-hosting provides more control and flexibility but demands significant operational effort.

#### Common Mistakes/Follow-Ups
Underestimating Operational Effort: Mistrusting self-hosted solutions to be less complex than they are.
Overlooking Cost Overheads: Assuming managed offerings will always be cost-effective without considering usage-based billing.
Ignoring Security Implications: Failing to adequately secure a self-hosted cluster, especially in cloud environments.

**Comparison:**

```
Resource	Self-Hosting	Managed Offering
Control	High	Low
Cost	Variable, High Initial Setup	Lower, Usage-Based Billing
Complexity	High (Ongoing)	Low (Initial)
```

**Code:**

```bash
# Example of a self-hosted Temporal configuration file
temporal-cluster:
  persistence:
    type: s3
    bucket_name: temporal-backup-bucket
    region: us-west-2
```

**Example input / output:**

```text
Input:
Start Workflow with ID 123

Output:
Workflow successfully started and running in Temporal cluster.
```

**Data:**

```json
{
  "persistence": {
    "type": "s3",
    "bucketName": "temporal-backup-bucket",
    "region": "us-west-2"
  }
}
```

**Flow:**

1. Self-Hosting
2. Control Over Stack
3. Cost of Resources
4. Complexity of Operations
5. Safety and Security
6. Managed Offering
7. Tight Integration
8. Cost-Effective
9. Ease of Use
10. Support from Provider

