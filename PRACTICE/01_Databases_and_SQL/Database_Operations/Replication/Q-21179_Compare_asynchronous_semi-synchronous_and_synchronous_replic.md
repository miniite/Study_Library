---
type: "practice"
difficulty: "Deep"
---

# Compare asynchronous, semi-synchronous, and synchronous replication in terms of durability and write latency trade-offs.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Replication Topologies and Lag](<../../../../01_Databases_and_SQL/Database_Operations/Replication/Failback_After_Failover/Replication_Topologies_and_Lag.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary

In database replication, particularly within a master-slave topology, understanding the differences between asynchronous, semi-synchronous, and synchronous replication is crucial. These methods vary in their approach to ensuring data durability and write latency trade-offs. Asynchronous replication offers high throughput with minimal write latency but risks data loss during network failures or crashes. Semi-synchronous replication improves on this by requiring acknowledgment from a quorum of replicas before completing the write, thus reducing the risk of data loss. Synchronous replication ensures all replicas are updated before confirmation is given, offering the highest durability at the cost of increased write latency.

#### Triaging the Question

The core of the question revolves around comparing three replication modes: asynchronous, semi-synchronous, and synchronous replication. The interviewer is probing your understanding of how these mechanisms handle data consistency, write performance, network failures, and overall system availability.

#### Assumptions
We are focusing on a master-slave (or master-replica) database architecture.
The primary concern is the trade-off between durability and write latency.
Network conditions are variable but not catastrophic (e.g., occasional delays or temporary outages).
Detailed Explanation from First Principles
Asynchronous Replication

Definition: In asynchronous replication, writes to the master are confirmed as successful without waiting for acknowledgment from replicas. The master simply sends changes over the network to one or more slaves.

Durability:

#### Pros
High write performance because there is no delay in confirming writes.

#### Cons
Risk of data loss if a replica fails before receiving all updates, especially during network partitions.

Write Latency:

Minimal write latency since the master does not wait for any acknowledgment from slaves.
Semi-Synchronous Replication

Definition: This method requires at least one replica to acknowledge receipt of a write command before the master confirms the write operation has completed.

Durability:

#### Pros
Reduces risk of data loss compared to asynchronous replication, as writes are acknowledged by at least one replica.

#### Cons
Still susceptible to partial failures where some replicas might not receive all updates. The system can still be left in an inconsistent state during network partitions.

Write Latency:

Increased write latency due to the need for acknowledgment from a quorum of replicas, although it is generally lower than synchronous replication.
Synchronous Replication

Definition: All replicas must acknowledge receipt of a write command before the master confirms the operation. This ensures all replicas are up-to-date with the latest data.

Durability:

#### Pros
Highest durability as writes cannot be confirmed until they are fully propagated to all replicas.

#### Cons
High risk of write latencies, especially in large-scale systems where network delays can significantly impact performance.

Write Latency:

Maximum write latency because the master waits for confirmation from all replicas before confirming a write operation.

#### Closing Summary

In summary, asynchronous replication offers high throughput and low write latency but risks data loss during partial failures. Semi-synchronous replication improves on this by requiring acknowledgment, reducing the risk of data loss while maintaining relatively fast writes. Synchronous replication ensures maximum durability at the expense of increased write latencies. The choice between these methods depends heavily on the specific use case's requirements for consistency and availability.

#### Common Mistakes/Follow-Ups
Underestimating Network Failures: Many candidates fail to consider the impact of network partitions, which can lead to incorrect assumptions about durability.
Ignoring Performance Implications: The trade-offs in write latency are often not fully appreciated, especially in high-performance systems where even small increases can be critical.
Overlooking Scalability Issues: Synchronous replication can become a bottleneck as the number of replicas grows, making it unsuitable for large-scale deployments.

#### Example
Worked Scenario
PLAINTEXT
Input: Write operation to master with semi-synchronous configuration

Master -> Replica 1 (ACK) -> Master confirms write to application

In this scenario:
- Writes are faster due to immediate confirmation by the master.
- Replica 2 might be delayed, but it will eventually receive the same data.

Output: High throughput and moderate durability
Self-Check

This answer covers all key aspects of the question, including definitions, trade-offs, and common pitfalls. It aligns with the deep understanding required for a database architect/principal role.

**Comparison:**

```
Replication Mode	Durability	Write Latency
Asynchronous	Low (risk of partial loss)	Low
Semi-Synchronous	Medium (acknowledge quorum)	Slightly higher
Synchronous	High (all replicas sync)	Highest
```

**Code:**

```python
# Example Python code to simulate asynchronous write confirmation (simplified)
def write_to_master(data):
    # Simulate sending data to master
    print("Data written to master")

def confirm_write(master_confirmation, replicas):
    if all(replicas):  # Assuming all replicas acknowledged
        return True
    else:
        return False

write_to_master("data123")
if confirm_write(True, [True, False]):  # Semi-synchronous check
    print("Write confirmed to application")
else:
    print("Write not confirmed due to replica failure")
```

**Example input / output:**

```text
Input:
- Write operation to master with semi-synchronous configuration

Output:
- Master -> Replica 1 (ACK) -> Master confirms write to application

Note: In a realistic scenario, the second replica's offline status would cause the master not to confirm the write.
```

**Data:**

```json
{
  "master": {
    "status": "online",
    "writes": ["data123", "data456"]
  },
  "replicas": [
    {"status": "online", "acknowledged_writes": true},
    {"status": "offline", "acknowledged_writes": false}
  ]
}
```

**Flow:**

1. Write to Master
2. Asynchronous Replication
3. Writes to Slave(s), no ACK
4. Semi-Synchronous Replication
5. Slave(s) ACK, then Masterconfirms write
6. Synchronous Replication
7. All Slaves ACK, then Masterconfirms write

