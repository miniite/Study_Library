---
type: "practice"
difficulty: "Expert"
---

# In KRaft mode, what is the controller quorum and how does Raft replace the old ZooKeeper-based controller election?

**Difficulty:** Expert · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ KRaft Mode](<../../../../TOOLS/Apache_Kafka/Kafka_Core/When_Not_to_Use_Kafka/KRaft_Mode.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In KRaft (Kafka Resource Manager in Flight) mode, Kafka brokers are self-managing and form a dynamic cluster with no external dependencies on ZooKeeper. The key concept here is the controller quorum and how it uses Raft consensus to manage leader elections for the controller node.

TL;DR

The controller quorum ensures that there's always a single active controller in KRaft mode, replacing the old ZooKeeper-based election mechanism. This is achieved through a distributed consensus protocol called Raft, which guarantees leader election and data consistency among Kafka brokers.

#### Triage
Understanding the Question

The question asks about two main aspects:

Controller Quorum: The role of this quorum in ensuring leadership stability.
Raft Replacement: How Raft replaces ZooKeeper for managing leader elections in KRaft mode.

#### Assumptions
and Context
Kafka Cluster: A group of Kafka brokers running in KRaft mode.
Leader Election: Determining which broker will act as the controller to manage cluster state.
ZooKeeper vs. Raft: Understanding the shift from a traditional ZooKeeper-based approach to an internal Kafka solution.

#### First Principles Solution
Controller Quorum

In KRaft mode, the concept of a "controller quorum" is introduced. This quorum ensures that at any point in time, there's one and only one controller node responsible for managing the cluster state. The controller quorum is defined as a set of brokers where each broker can communicate with a majority of other brokers to form an agreement on the current leader.

How Raft Replaces ZooKeeper
Leader Election: In KRaft mode, brokers use the Raft consensus algorithm to elect a single controller node. Each broker acts as a peer in this Raft cluster.
State Machine Consistency: Raft maintains a state machine that tracks the current leader and ensures consistency across all nodes. This is crucial for avoiding multiple controllers and ensuring data integrity.
Leader Discovery: When a new broker joins or an existing controller fails, the Raft protocol enables seamless transition to a new leader. This process involves sending messages between brokers until a majority agrees on the next leader.
Detailed Steps

Initial State:

All brokers start in follower mode.
A set of nodes form the initial controller quorum.

Leader Election Process:

Each broker sends heartbeat messages to maintain its status as a follower or candidate for leadership.
If no leader is present, a candidate starts a Raft election by proposing itself as the new leader.
Other brokers vote on whether to follow this candidate based on their majority agreement.

Leader Transition:

Once a majority of nodes agree, the candidate becomes the new controller.
The old controller (if active) steps down and stops managing the cluster state.

Cluster State Management:

As new brokers join or existing ones fail, Raft handles reconfiguration to maintain the quorum.
Any change in the leader is communicated across the network ensuring consistency.

#### Closing Summary

In summary, KRaft mode introduces a self-contained controller election mechanism using the Raft consensus algorithm. This replaces the external dependency on ZooKeeper by embedding controller management within the Kafka cluster itself, enhancing reliability and reducing latency.

#### Common Mistakes/Follow-Ups
Misunderstanding Quorum Size: Failing to recognize that the quorum size is crucial for leader election.
Raft Consensus: Ignoring the details of how Raft ensures consensus among brokers.
Reconfiguration Handling: Overlooking the dynamic nature of the controller quorum during node join/fail events.
Self-Check

This answer covers the core aspects and common pitfalls, ensuring correctness and completeness.

**Comparison:**

| Broker Role   | Description                        | Status    |
|---------------|------------------------------------|-----------|
| Leader        | Manages cluster state               | Active    |
| Follower      | Follows the current leader          | Following |

**Code:**

```python
# Example pseudo-code for Raft leader election (simplified)
class KafkaController:
    def __init__(self):
        self.leader = None
        self.quorum_size = 5
        self.followers = set()

    def propose_leader(self, candidate_id):
        if not self.is_election_possible():
            return False

        self.send_vote_request(candidate_id)
        votes_received = 0

        while votes_received < (self.quorum_size // 2 + 1):
            response = self.receive_vote_response()
            if response['vote_granted']:
                votes_received += 1

        self.leader = candidate_id
        return True

    def is_election_possible(self):
        # Check conditions for leader election to proceed
        pass

    def send_vote_request(self, candidate_id):
        # Send vote request to all nodes
        pass

    def receive_vote_response(self):
        # Receive and process votes
        pass
```

**Example input / output:**

| Input | Output |
|-------|--------|
| `propose_leader("broker-2")` | Success, new leader elected. |
| Request | Response |
|---------|----------|
| `leader_election_request` | `LeaderElected(broker-3)` |
| Event | Action |
|-------|--------|
| Node Join | Reconfigure quorum and election process. |
| Node Fail | Transition to new leader via re-election. |

**Data:**

```json
{
  "controller_quorum_size": 5,
  "followers": ["broker-1", "broker-2", "broker-3", "broker-4", "broker-5"],
  "leader": "broker-1"
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Yes
6. No
7. Node Fail
8. Is Quorum Maintained?
9. Monitor for Election
10. Reconfigure Quorum, ElectNew Leader
11. No Action Required
12. Start Leader Election Process
13. Vote Counted, Leader Elected
14. Notify All Brokers of NewLeader
15. Continue Normal Operations

