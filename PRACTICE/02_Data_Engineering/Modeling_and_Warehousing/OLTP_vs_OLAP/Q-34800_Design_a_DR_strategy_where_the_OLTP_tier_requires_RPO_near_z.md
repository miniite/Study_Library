---
type: "practice"
difficulty: "Deep"
---

# Design a DR strategy where the OLTP tier requires RPO near zero across regions but the OLAP tier can be fully reconstructed from source — how does that asymmetry shape your replication topology and cost?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Feeding OLAP from OLTP](<../../../../02_Data_Engineering/Modeling_and_Warehousing/OLTP_vs_OLAP/Separating_OLTP_and_Analytics/Feeding_OLAP_from_OLTP.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a disaster recovery (DR) strategy with an OLTP tier requiring near-zero RPO across regions, while the OLAP tier can be fully reconstructed from source, we must carefully balance performance, availability, and cost. The key is to understand that the OLTP tier demands real-time data consistency, whereas the OLAP tier allows for delayed updates. This asymmetry shapes our replication topology by focusing on high-speed, low-latency replication for critical transactional data while allowing more relaxed, periodic synchronization for analytical data.

The core idea here is to use a multi-tiered approach: one focused on near-zero RPO with local synchronous replication and another that can leverage asynchronous or snapshot-based replication. Let's break down the solution step by step:

#### Triaging the Question
RPO Requirement: Near-zero RPO means minimal data loss in case of an outage.
Data Types:
OLTP (Online Transaction Processing): High transaction volume, real-time consistency required.
OLAP (Online Analytical Processing): Low transaction volume, can tolerate some delay.

#### Assumptions
Multiple regions with potential outages.
Cost optimization is a key factor.
High availability for the OLTP tier is critical.
Step-by-Step Solution
1. OLTP Tier Design

For the OLTP tier, we need to ensure minimal RPO and strong consistency:

Synchronous Replication: Set up synchronous replication between primary and secondary regions. This ensures that data written in one region is immediately acknowledged as committed.
Local Backup: Implement local backups at each region for additional protection against hardware failures.
2. OLAP Tier Design

For the OLAP tier, a different approach can be taken:

Asynchronous Replication or Snapshots: Use asynchronous replication to periodically update secondary nodes from the primary node. This allows for lower costs and less network load.
Periodic Full Reconstruction: Develop a process that can fully reconstruct the OLAP data from source in case of a failure.
3. Topology Design

A possible topology could involve:

Primary Region: High availability, high transaction volume.
Secondary Regions (Near and Far): Lower latency for near regions, higher latency but more cost-effective for far regions.
Primary Region
Synchronous Replication
Local Backup
Near Secondary Region
Asynchronous Replication
Periodic Full Reconstruction
Far Secondary Region
Snapshot-Based Replication
4. Cost Considerations
Primary and Near Regions: Higher cost due to synchronous replication, but crucial for high availability.
Far Regions: Lower cost with asynchronous replication or snapshots.

#### Closing Summary

This design ensures that the OLTP tier remains highly available with minimal RPO, while the OLAP tier leverages less expensive replication techniques. The key is to balance these requirements by carefully choosing the right topology and replication methods for each tier.

#### Common Mistakes/Follow-Ups
Ignoring Cost: Failing to consider cost implications can lead to overly complex solutions.
Over-replication: Synchronous replication across all regions might be unnecessary, increasing costs without providing additional value.
Underestimating Reconstruction Time: Assuming that OLAP data can always be fully reconstructed within a reasonable time frame.

**Comparison:**

| Region | Replication Mode | Cost |
|--------|------------------|------|
| Primary| Synchronous      | High |
| Near  | Asynchronous     | Low  |
| Far   | Snapshot-Based   | Med  |

**Code:**

```sql
-- Example SQL for configuring synchronous and asynchronous replication

-- Synchronous Replication Setup
CREATE DATABASE INSTANCE 'primary-instance';
CREATE DATABASE INSTANCE 'secondary-near-instance';
CREATE DATABASE INSTANCE 'secondary-far-instance';

SET REPLICATION MODE 'Synchronous' ON 'primary-instance';
SET REPLICATION PARTNER 'secondary-near-instance', 'secondary-far-instance';
```

**Example input / output:**

```text
Input:
- Transactional data (OLTP) from Primary Region to Near and Far Regions.
- Periodic updates for OLAP data.

Output:
- Synchronous replication ensuring minimal RPO for OLTP.
- Asynchronous or snapshot-based replication for cost optimization in OLAP.
```

**Data:**

```json
{
  "regions": [
    {
      "name": "Primary",
      "replication": "Synchronous"
    },
    {
      "name": "Near Secondary",
      "replication": "Asynchronous",
      "cost": "Low"
    },
    {
      "name": "Far Secondary",
      "replication": "Snapshot-Based",
      "cost": "Medium"
    }
  ]
}
```

**Flow:**

1. Primary
2. Synchronous
3. Near Secondary
4. Asynchronous
5. Far Secondary
6. Periodic Full Reconstruction
7. OLAP Data Rebuild

