---
type: "practice"
difficulty: "Expert"
---

# Explain how KRaft changes your operational runbook for controller failover, and what new failure modes the metadata quorum introduces versus ZooKeeper.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ KRaft Mode](<../../../../TOOLS/Apache_Kafka/Kafka_Core/When_Not_to_Use_Kafka/KRaft_Mode.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

KRaft is a significant update to Apache Kafka that replaces the traditional ZooKeeper-based coordination with its own leader election mechanism. This change impacts how you handle controller failover, introducing new failure modes and operational considerations.

#### Triaging the Question

The question focuses on understanding the operational changes brought about by KRaft in the context of controller failover in Kafka clusters. Specifically, it delves into how KRaft alters the runbook for failover scenarios and highlights new potential failure points compared to the previous ZooKeeper-based setup.

#### Assumptions
Current Knowledge: You are familiar with both traditional ZooKeeper-based Kafka operations and the basics of KRaft.
Operational Context: The discussion centers on how a backend SDE-3/Architect would manage controller failover in a Kafka cluster using KRaft.

#### First Principles Solution
Controller Failover under Traditional ZooKeeper:

In traditional ZooKeeper-based Kafka clusters, the controller's role is managed by external ZooKeeper nodes. During a failover, you typically follow these steps:

Monitor: Monitor the state of ZooKeeper and detect when a leader controller fails.
Failover Triggering: A new Kafka broker can trigger a failover based on the absence of a heartbeat from the current controller.
Election Process: The remaining controllers (or Kafka brokers) run an election process using ZooKeeper to elect a new leader.
Configuration Update: Update the cluster configuration with the new controller details.
KRaft for Controller Failover:

Kafka's KRaft model introduces significant changes, centralizing the controller role within the Kafka nodes themselves and eliminating the need for external ZooKeeper nodes. The operational steps now involve:

Health Monitoring: Each Kafka node monitors its own state and checks if it should take on the role of the controller.
Leader Election: A leader election is managed internally among the Kafka brokers, using Raft consensus to ensure a single leader is elected.
Metadata Update: The new leader updates the metadata to reflect its leadership status, ensuring all nodes are aware of the current leader.
Failure Modes Introduced by KRaft:

Leader Election Disruption:

With ZooKeeper, if an external node fails, you can still have a functional cluster as long as there is enough quorum.
In KRaft, if multiple Kafka brokers fail simultaneously and cannot form a Raft majority, the entire cluster may enter a maintenance state.

Data Consistency Issues:

ZooKeeper's distributed nature helps in ensuring data consistency across nodes even during partial failures.
KRaft relies on internal consensus mechanisms, which can introduce complexity if not all nodes are in sync due to network partitions or node outages.

Complexity of Maintenance:

While ZooKeeper simplifies the operational runbook by abstracting leader election and state management, KRaft requires deeper knowledge of Raft protocols and Kafka's internal mechanisms.
This can lead to more complex diagnostics and maintenance tasks, especially in large clusters with high availability requirements.

Failure Detection Latency:

ZooKeeper has built-in mechanisms for quick detection of leader failures due to its distributed nature.
KRaft’s reliance on Raft consensus can introduce additional latency in detecting and responding to controller failures, which could impact critical operations like rebalancing topics or managing partitions.

#### Closing Summary

KRaft significantly alters the operational landscape for Kafka cluster management. While it simplifies certain aspects by centralizing leader election within the Kafka nodes themselves, it introduces new failure modes and complexity that need careful consideration during maintenance and troubleshooting. Understanding these changes is crucial for any backend SDE-3 or Architect to ensure robust operation of Kafka clusters.

#### Common Mistakes/Follow-Ups
Overlooking Internal Consensus Delays: Failing to account for potential delays in Raft consensus can lead to unexpected downtime.
Misinterpreting Failure Modes: Misunderstanding the impact of leader election disruptions on cluster operations can result in suboptimal failover strategies.
Ignoring Data Synchronization Issues: Not addressing data consistency issues during leader transitions can lead to operational problems.
Triaging Common Follow-Ups:
How does KRaft handle network partitions?
What are the best practices for monitoring a Kafka cluster running with KRaft?
How do you ensure high availability in case of Raft-based leader elections failing?

**Comparison:**

| Kafka Node | Role     | Status         |
|------------|----------|----------------|
| broker-01  | Leader   | Down           |
| broker-02  | Follower | Active         |
| broker-03  | Follower | Active         |

**Code:**

```python
# Example pseudo-code for basic KRaft controller failover detection and response
def detect_controller_failure():
    # Check if current controller is still active
    if not is_controller_alive(current_leader):
        # Trigger a leader election
        new_leader = run_leader_election()
        return new_leader
    return None

def run_leader_election():
    # Use Raft protocol to elect a new leader among Kafka brokers
    election_result = raft_consensus(elect_leader=True)
    if election_result.success:
        return election_result.leader
    else:
        raise LeaderElectionFailedException()
```

**Example input / output:**

```text
Input:
Monitor Kafka cluster for controller failure

Output:
- If no leader is detected, trigger internal Raft-based election.
- Elect new leader if quorum achieved.
- Update metadata to reflect new leadership.

Dry-run:
Given a cluster with brokers [broker-01, broker-02, broker-03] and currentLeader as broker-01,
check for failover. Since broker-01 is down, run_leader_election() returns broker-02.
Update status: {"currentLeader": "broker-02", "status": "healthy"}
```

**Data:**

```json
{
  "cluster": {
    "currentLeader": "broker-01",
    "brokers": ["broker-01", "broker-02", "broker-03"],
    "quorum": 2,
    "status": "healthy"
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Monitor Kafka Cluster
6. Is Current Leader Alive?
7. Continue Normal Operations
8. Trigger Internal Raft-basedElection
9. Quorum Achieved?
10. Elect New Leader, UpdateMetadata
11. Enter Maintenance Mode

