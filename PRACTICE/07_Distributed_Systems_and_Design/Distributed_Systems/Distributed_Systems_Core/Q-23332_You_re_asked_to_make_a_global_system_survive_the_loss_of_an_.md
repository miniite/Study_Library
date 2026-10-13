---
type: "practice"
difficulty: "Expert"
---

# You're asked to make a global system survive the loss of an entire cloud region. Compare active-active, active-passive, and pilot-light topologies at the RTO/RPO/cost frontier, and describe how you'd actually rehearse a region evacuation so the plan isn't fiction on the day you need it.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Sync vs Async Replication](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Sync_vs_Async_Replication.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

To ensure our global system can survive the loss of an entire cloud region, we must compare active-active, active-passive, and pilot-light topologies based on Recovery Time Objective (RTO), Recovery Point Objective (RPO), and cost. We'll then outline a detailed rehearsing process to validate these strategies.

#### Triaging the Question

The core of this question is understanding and comparing different failover strategies for distributed systems, specifically in the context of cloud regions. The interviewer wants to know your ability to evaluate trade-offs between RTO/RPO and costs while ensuring system resilience. Assumptions include:

Each region has its own set of data centers.
We need a strategy that can handle the loss of an entire region without significant downtime or data loss.

#### First Principles Solution
Active-Active Topology

RTO: Minimal, ideally zero downtime as writes are replicated in real-time to other regions. RPO: Zero if replication is immediate. However, network latency may introduce a small RPO. Cost: High due to the need for multiple replicas and high-speed data synchronization mechanisms. Rehearsal Plan: Simulate an outage by isolating one region temporarily while maintaining read/write operations in others. Verify real-time replication and failover.

Active-Passive Topology

RTO: Typically higher than active-active, but still quick to switch if the passive region is ready. RPO: Higher as writes are not immediately replicated. They will be replicated over time. Cost: Lower compared to active-active since only one region is actively handling writes. Rehearsal Plan: Simulate an outage by shutting down a region for maintenance or testing purposes, then ensure that the active region can seamlessly take over.

Pilot-Light Topology

RTO: Higher due to potential need for failover operations. Can be managed with pre-emptive scaling of passive regions. RPO: Similar to active-passive but may depend on how frequently pilot-light nodes are updated. Cost: Lower as it involves only a small number of nodes in standby mode ready to fail over quickly. Rehearsal Plan: Regularly test the failover process by setting up pilot-light nodes and ensuring they can handle writes efficiently.

#### Closing Summary

For resilience, the active-passive topology strikes a balance between RTO, RPO, and cost. It allows for real-time data synchronization without the high overhead of multiple active regions, making it suitable for most enterprise applications where some level of delay in write operations is acceptable but quick failovers are essential.

#### Common Mistakes
and Follow-Ups
Mistake: Overlooking the importance of RPO in pilot-light topologies.

#### Follow-Up
Discussing how to balance cost with RTO/RPO requirements by considering hybrid approaches or cloud services that offer managed disaster recovery solutions.

**Code:**

```
MERMAID
Copy
flowchart TD
    A[Region 1 Active] -->|Writes| B[Region 2 Passive]
    C[Isolate Region 1] --> D{RTO OK?} --> E[Revert Changes] || F[Proceed with Failover Plan]
```

**Example input / output:**

```text
**Input**
- Simulate an outage by isolating Region 1.
- Monitor replication and failover processes.

**Output**
- Ensure seamless transition to Region 2 with minimal downtime.
- Validate data consistency across regions.
- Review performance metrics for new active region.
```

**Data:**

```json
{
  "regions": [
    {"id": "R1", "status": "active"},
    {"id": "R2", "status": "passive"}
  ],
  "failoverPlan": {
    "testDate": "2023-10-15",
    "isolationProcedure": ["Shutdown R1", "Monitor Replication", "Switch Traffic to R2"],
    "recoveryChecklist": [
      {"item": "Data Consistency Check"},
      {"item": "Performance Metrics Review"}
    ]
  }
}
```

