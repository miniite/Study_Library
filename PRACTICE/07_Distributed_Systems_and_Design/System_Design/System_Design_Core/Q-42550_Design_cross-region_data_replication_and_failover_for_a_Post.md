---
type: "practice"
difficulty: "Deep"
---

# Design cross-region data replication and failover for a Postgres-backed service with an RPO of seconds and an RTO of minutes. What replication mode, and how do you avoid promoting a stale replica?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Primary-replica Databases](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Clickstream_Counting_Pipeline/Primary-replica_Databases.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In designing a cross-region data replication and failover system for a Postgres-backed service with an RPO (Recovery Point Objective) of seconds and an RTO (Recovery Time Objective) of minutes, we need to carefully balance performance, consistency, fault tolerance, and cost. The core challenge is to ensure that the primary region remains operational while providing seamless failover in case of a failure, all within the constraints of RPO and RTO.

#### Triage
Understanding the Question

The question is asking for a system design that ensures minimal data loss (RPO < 1 second) during a failure and rapid recovery (RTO minutes). We need to choose an appropriate replication mode and implement failover mechanisms to avoid promoting stale replicas. The key aspects are:

Replication Mode: How data is replicated between regions.
Failover Mechanisms: Ensuring the primary region remains available while quickly promoting a replica as the new primary.

#### First Principles Solution
Replication Mode: Multi-Master Asynchronous Replication

To achieve an RPO of seconds, we need to use multi-master asynchronous replication. This mode allows writes in both regions without requiring immediate consistency across all replicas. PostgreSQL supports this through its streaming replication capabilities combined with logical decoding. The primary region will handle write requests and stream these changes to the secondary regions asynchronously.

Failover Mechanisms: Leader Election and Consensus Protocols

To avoid promoting a stale replica, we need robust failover mechanisms that ensure only the most recent state is promoted as the new primary. This can be achieved using consensus protocols like Paxos or Raft, which are designed for distributed systems with strong consistency requirements.

Detailed Solution
Multi-Master Asynchronous Replication
Primary Region: The main database node where all write requests originate.
Secondary Regions: Multiple replicas in different regions that asynchronously stream changes from the primary region via logical decoding.
Data Streaming: Data is replicated using PostgreSQL’s logical replication feature, which allows for custom decoders to capture and transmit only necessary data changes.

#### Cons
ensus Protocols
Leader Election: Use a consensus protocol like Raft or Paxos to elect a leader among the secondary replicas. The leader will be responsible for determining when to promote itself as the new primary.

#### Cons
istency Checks: Implement periodic consistency checks between the primary and secondary replicas. This ensures that only the most recent state is promoted.
Automatic Failover: If the primary region fails, the Raft/Raft protocol leader can automatically take over and promote itself to the primary role.
Handling Conflicts

In multi-master environments, conflicts may arise due to concurrent writes in different regions. To handle these:

Conflict Resolution Logic: Implement conflict resolution logic within the application or at the database level using techniques like MVCC (Multi-Version Concurrency Control).
Manual Conflict Resolution: For critical updates, provide a manual conflict resolution mechanism where human intervention is required to resolve conflicts.
Monitoring and Alerts
Health Checks: Continuously monitor the health of all replicas.
Alerts: Set up alerts for any anomalies or failures in the primary region or secondary replicas.

#### Closing Summary

By leveraging multi-master asynchronous replication with PostgreSQL and employing consensus protocols like Raft for leader election, we can achieve an RPO of seconds and an RTO of minutes. This design ensures minimal data loss and rapid recovery while maintaining strong consistency across regions.

#### Common Mistakes
and Follow-Ups
Mistake: Choosing synchronous replication instead of asynchronous replication leads to high RTOs as all writes must wait for acknowledgments.

#### Follow-Up
Discuss the trade-offs between synchronous and asynchronous replication in terms of performance, reliability, and cost. Highlight how multi-region setups benefit from asynchronous replication due to network latency.

**Comparison:**

```
Region	Role	Status
us-west-2	Primary	Active
eu-central-1	Secondary	Idle
ap-southeast-1	Secondary	Idle
```

**Code:**

```python
# Example of a simple Raft leader election script
import time

def elect_leader(replicas):
    # Leader election logic using Paxos/Raft consensus protocol
    pass
```

**Example input / output:**

```text
Input:
- A write request to the primary region.
- A failure in the primary region.

Output:
- Write request forwarded to secondary replicas via logical decoding.
- Raft protocol leader identified and elected.
- New primary replica promoted, ensuring minimal RPO and RTO.
```

**Data:**

```json
{
  "primary_region": "us-west-2",
  "secondary_regions": ["eu-central-1", "ap-southeast-1"],
  "replication_mode": "multi-master asynchronous",
  "consensus_protocol": "Raft"
}
```

**Flow:**

1. Primary Region
2. Failure?
3. Replication to SecondaryRegions
4. Leader Election via Raft
5. New Primary Identified?
6. Promote New Primary
7. RTO Minimized

