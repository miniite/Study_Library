---
type: "practice"
difficulty: "Expert"
---

# How does the choice of failover strategy change for a database backing regulated financial reporting where auditors require provable no-data-loss? What evidence do you need to produce?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ HA Failover Design](<../../../../01_Databases_and_SQL/Database_Operations/High_Availability/HA_Failover_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a highly-regulated financial environment, ensuring data integrity and proving the absence of data loss during failovers is paramount. The choice of failover strategy directly impacts how we meet these stringent requirements. For this scenario, we need to consider both performance and compliance, as auditors demand provable no-data-loss guarantees.

#### Triaging the Question

The core question revolves around selecting a database failover strategy that ensures zero data loss in a highly-regulated financial environment. The key aspects are:

Compliance Requirements: Auditors require concrete evidence of no data loss.
Failover Strategy: Determining which strategy (e.g., synchronous vs. asynchronous) to use and how it aligns with regulatory needs.

#### Assumptions
We have a high-availability requirement due to the critical nature of financial reporting.
There are strict compliance regulations that mandate zero data loss during failovers.
The database system is part of a larger infrastructure that includes hardware, network, and storage components.

#### First Principles Solution
Synchronous Replication

Synchronous replication ensures that all nodes in the cluster maintain consistent state. Data writes to the primary node only succeed after they are also written successfully to the secondary node(s). This guarantees no data loss since any transaction committed on the primary is guaranteed to be replicated before it returns a success.

Advantages:

Ensures zero data loss.
Provides strong consistency guarantees, critical for financial transactions where even a single lost transaction can have significant implications.

Disadvantages:

Can introduce latency, as the write operation must wait for confirmation from all replicas before returning.
May require more infrastructure resources (e.g., additional storage and network capacity) to support synchronous writes.
Asynchronous Replication

Asynchronous replication allows the primary node to return a success response even if the secondary nodes are still processing the data. This can significantly reduce latency but introduces the risk of data loss during failovers, which must be carefully managed.

Advantages:

Lower latency and better performance.
More scalable, as it requires less infrastructure to support asynchronous writes.

Disadvantages:

Higher risk of data loss if a node fails before the secondary nodes have processed the transactions.
Compliance with zero data loss becomes challenging without additional mechanisms like log shipping or periodic manual failovers.
Implementing Zero Data Loss

Given the compliance requirements, synchronous replication is the preferred choice. However, to meet performance and cost trade-offs, we can implement a hybrid approach:

Hybrid Approach: Synchronous Primary Replication + Asynchronous Secondary Replication
Primary Node: Uses synchronous write operations to ensure data consistency.
Secondary Nodes (Read-Only): Use asynchronous replication for handling read requests, reducing latency.

This setup leverages the strengths of both strategies:

Synchronous Write Operations on the primary node ensures zero data loss.
Asynchronous Read Operations on secondary nodes provide low-latency reads without compromising data integrity.
Evidence and Compliance

To satisfy auditors' requirements for no-data-loss, we need to implement comprehensive logging and verification mechanisms:

Transaction Logs: Maintain detailed logs of all transactions that occur during normal operations and failovers.
Checkpoint Mechanisms: Use checkpointing to ensure that the state of the database is consistent before a failover occurs.
Audit Trails: Implement audit trails that capture every change in the system, including failed attempts at failovers.
Regular Audits: Conduct regular audits and reviews to verify the integrity of transaction logs and the consistency of the database across nodes.

#### Common Mistakes
and Follow-Ups
Ignoring Latency Trade-offs: Failing to balance performance with data consistency can lead to suboptimal systems that may not meet regulatory requirements.
Overlooking Recovery Strategies: Not having a robust recovery strategy in place, such as warm standby or log shipping, increases the risk of data loss during unexpected failures.
Insufficient Logging and Monitoring: Lack of comprehensive logging and monitoring mechanisms can make it difficult to prove the integrity of the database state.

#### Closing Summary

In conclusion, for a regulated financial reporting environment where auditors require provable no-data-loss guarantees, synchronous replication is the primary strategy. However, hybrid approaches that combine synchronous writes with asynchronous reads can balance performance and compliance needs effectively. Comprehensive logging, checkpointing, and regular audits are essential to meet regulatory requirements.

**Comparison:**

```
Node	Role	Replication Mode
A	Primary	Synchronous
B, C	Secondary	Asynchronous
```

**Code:**

```
MERMAID
Copy
graph TD;
    A[Primary Node] --> B1[Synchronous Write]
    A --> C2[Asynchronous Read]
    B1 --> D1[Log Transaction]
    C2 --> E2[Check Consistency]
```

**Example input / output:**

| Input | Output |
|-------|--------|
| Write Transaction Request (Synchronous) | Success Response with Log Entry |
| Read Request (Asynchronous)            | Data Retrieved from Secondary Node |

**Data:**

```yaml
# Example Log Entry
- timestamp: 2023-10-01T14:30:00Z
  transaction_id: 1234567890
  operation: write
  data: {"account": "ACCT-001", "amount": 100.0}
  node: primary

# Example Consistency Check
- timestamp: 2023-10-01T14:35:00Z
  result: consistent
  node: secondary_1
```

**Flow:**

1. Yes
2. No
3. Primary Write
4. Log Transaction
5. Wait for Acknowledgmentfrom All Secondaries
6. Success?
7. Return Success Response
8. Retry or Fail
9. Fallback to Asynchronous Read

