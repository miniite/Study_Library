---
type: "practice"
difficulty: "Expert"
---

# State the FLP impossibility result in plain terms. If consensus is impossible in a fully asynchronous system with even one faulty node, how do Raft/Paxos work in practice?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Quorums and Consensus](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Quorums_and_Consensus.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

The FLP Impossibility Result states that it's impossible to achieve fault-tolerant consensus in an entirely asynchronous distributed system where at least one process may fail arbitrarily. In plain terms, if nodes can fail unpredictably and messages can be delayed or lost indefinitely, there’s no reliable way for all processes to agree on a single value without potentially getting stuck in an infinite loop.

Now, let's dive into how Raft and Paxos work despite this limitation.

#### Triaging the Question

The question is probing your understanding of distributed systems’ inherent challenges, specifically the FLP Impossibility Result. The key assumptions here are:

You need to explain why consensus cannot be achieved in an asynchronous system.
You should describe the practical approaches used by Raft and Paxos that circumvent this theoretical impossibility.

#### First Principles Solution
FLP Impossibility Result: Why It Matters

In a fully asynchronous distributed system, nodes can fail at any time and messages may never arrive. This unpredictability makes it impossible to guarantee consensus because:

Leader Election: If no leader is elected, processes might keep proposing values without agreement.
Message Delays: Messages that need to reach other nodes for agreement might be delayed indefinitely, preventing timely responses.
Practical Approaches: Raft and Paxos

Despite these challenges, both Raft and Paxos employ strategies to achieve consensus in practice:

Raft (Appendix A)

Leader Election: Raft introduces a leader election mechanism where one node is elected as the leader. The leader coordinates the replication of log entries across all nodes.
Term-Based State Machine: Raft uses terms to manage state transitions and ensure that only one leader exists at any given time, even if a previous leader has failed or stepped down.
Heartbeats and Timeout Mechanisms: A leader continuously sends heartbeats to other nodes. If a node doesn’t receive a heartbeat within its timeout period, it considers the current leader faulty and starts an election for a new leader.

Paxos (Agreement in the Presence of Faults)

Proposal and Acceptance Rounds: Paxos splits the consensus problem into two phases: proposal and acceptance.
Proposer Phase: Proposers propose values, and acceptors must choose one value to accept.
Acceptor Phase: Acceptors agree on a single value by maintaining their current best proposal.
Majority Agreement: To reach agreement, a majority of nodes must agree. This ensures that even if some nodes fail, the remaining nodes can still decide on a value.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, while the FLP Impossibility Result proves that consensus is unachievable in an entirely asynchronous system with arbitrary failures, practical solutions like Raft and Paxos use mechanisms such as leader election, term-based state machines, heartbeat timeouts, proposal/acceptance rounds, and majority agreement to circumvent these limitations.

#### Common mistakes
include misunderstanding the exact nature of FLP or assuming that Raft/Paxos solve all potential issues without understanding their trade-offs. For example, while Raft is simpler and more intuitive, it may not be as efficient in terms of network communication compared to Paxos, which can handle a larger number of nodes but requires more complex logic.

#### Follow-ups
might include discussing the specific failure modes each algorithm handles or comparing their performance metrics under different network conditions.

**Comparison:**

| Raft vs. Paxos | Raft | Paxos |
|---------------|------|-------|
| Complexity    | Simpler | More Complex |
| Performance   | Lower Throughput | Higher Throughput |
| Failure Modes | Timeout-based Election | Proposal/Acceptance Rounds |

**Code:**

```
MERMAID
Copy
graph TD;
    A[Leader Election] --> B{Timeout?};
    B -- No --> C[Continue Heartbeats];
    B -- Yes --> D[New Leader Elected];
    C --> E[Proposer Phase];
    E --> F[Acceptor Phase];
    F --> G[Majority Agreement Reached];
    D --> H[Heartbeats Restarted]
```

**Example input / output:**

```text
Input:
- Raft nodes start heartbeats and check for timeouts.
- Paxos nodes receive proposals and agree on a value.

Output:
- Raft elects a new leader if no heartbeat is received.
- Paxos reaches agreement if a majority of acceptors agree on the same proposal.
```

**Data:**

```json
{
  "raft": {
    "leader_term": 1,
    "last_log_index": 50,
    "current_index": 200,
    "heartbeat_interval": 500
  },
  "paxos": {
    "proposals": [
      {"value": "a", "accepted": true, "term": 3},
      {"value": "b", "accepted": false, "term": 4}
    ],
    "acceptors": ["node1", "node2", "node3"],
    "majority": 2
  }
}
```

**Flow:**

1. No
2. Yes
3. Node
4. Is Leader?
5. Start Heartbeat
6. Maintain Leadership
7. Timeout Occurred
8. New Election Initiated
9. Continue Normal Operation

