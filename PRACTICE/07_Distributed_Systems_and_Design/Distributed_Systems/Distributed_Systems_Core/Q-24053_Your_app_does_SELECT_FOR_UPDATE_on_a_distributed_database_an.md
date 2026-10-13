---
type: "practice"
difficulty: "Deep"
---

# Your app does SELECT ... FOR UPDATE on a distributed database and latency is 4x a single-node baseline. Explain how the distributed lock and the consensus round for the write intent add that cost and when you'd avoid pessimistic locking here.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Distributed Counters and Consistency](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Joins_in_MPP_Systems/Distributed_Counters_and_Consistency.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of high latency in SELECT ... FOR UPDATE operations on a distributed database, we need to understand how distributed locks and consensus rounds contribute to this overhead. Let's break down the problem step-by-step from first principles.

In a distributed system, each node maintains its own state, which can lead to inconsistencies if not managed properly. The SELECT ... FOR UPDATE operation acquires an exclusive lock on rows that might be updated in subsequent transactions. This is known as pessimistic locking. When this operation is performed across multiple nodes, it involves coordination and communication overhead.

#### Triage

The question is essentially asking us to analyze the performance impact of distributed locks and consensus rounds in a distributed database environment. We need to consider:

Distributed Locks: These ensure that only one transaction can update certain rows at a time.

#### Cons
ensus Rounds: This refers to achieving agreement among nodes on which updates should proceed.

#### Assumptions
The baseline performance is for a single-node setup, where the SELECT ... FOR UPDATE operation completes quickly without any network latency or distributed coordination overhead.
We are dealing with a distributed database system where transactions need to be coordinated across multiple nodes to ensure consistency.

#### First-Principles Solution

Distributed Locks: In a distributed system, acquiring a lock on rows that may be updated involves communication between the node performing the read and other nodes that might also update these rows. This requires setting up a distributed lock mechanism, which introduces latency due to network delays.

#### Cons
ensus Rounds: To ensure agreement among nodes, a consensus protocol (like Raft or Paxos) is used. Each transaction involving SELECT ... FOR UPDATE needs to go through several rounds of communication and decision-making processes. These rounds involve leader election, voting, and message exchanges between nodes, which add significant latency.

Detailed Breakdown

Read Phase:

The node performing the read operation sends a request to other nodes to check if any pending writes might affect the rows being selected.
Each node needs to respond with their state, which involves sending lock information and possibly waiting for responses from other nodes.

#### Cons
ensus Rounds:

A consensus protocol is triggered to ensure that all nodes agree on the outcome of the transaction. This involves multiple rounds where nodes send messages to each other.
Each round can involve leader election, where a node is chosen to propose the operation and gather votes from others.

Write Phase:

Once consensus is reached, the node can proceed with updating the rows if no conflicts are detected.
The updated data needs to be propagated to all nodes, which again involves network communication delays.

#### Closing Summary

The latency increase in SELECT ... FOR UPDATE operations on a distributed database is primarily due to the overhead of setting up and managing distributed locks and consensus rounds. These mechanisms ensure consistency but come at a cost, especially as the number of nodes grows or if network latency increases.

#### Common Mistakes
and Follow-Ups
Mistake 1: Assuming that the baseline performance can be directly translated to the distributed setup without considering additional overhead.

#### Follow-Up
Question: In what scenarios might you prefer optimistic locking over pessimistic locking, given this increased cost?

**Comparison:**

| Phase | Time (ms) |
|-------|----------|
| Read  | 2.5      |
| Consensus Round | 180     |
| Write  | 3.5      |

**Code:**

```
MERMAID
Copy
sequenceDiagram
    participant NodeA as "Node A"
    participant NodeB as "Node B"
    participant NodeC as "Node C"

    NodeA ->> NodeB: Request for lock state
    NodeB -->> NodeA: Lock state response

    NodeA ->> NodeC: Request for lock state
    NodeC -->> NodeA: Lock state response

    NodeA ->> NodeB: Propose update
    NodeB -->> NodeA: Vote request
    NodeA ->> NodeB: Vote
    NodeB -->> NodeA: Acknowledge vote

    NodeA ->> NodeC: Propose update
    NodeC -->> NodeA: Vote request
    NodeA ->> NodeC: Vote
    NodeC -->> NodeA: Acknowledge vote

    NodeA ->> NodeB: Update rows
```

**Example input / output:**

```text
Input (Single-node baseline):
- NodeA performs SELECT ... FOR UPDATE on rows R1 and R2.
- Transaction completes in ~0.5ms.

Output:
- Network latency between nodes is 1ms each way.
- Consensus round takes 3 rounds with a timeout of 50ms per round.

Input (Distributed setup):
- NodeA performs SELECT ... FOR UPDATE on rows R1 and R2.
- Lock state requests from other nodes take 4ms to receive.
- Consensus rounds involve multiple message exchanges, each taking ~60ms due to network latency.
```

**Data:**

```yaml
# Example consensus round data for Paxos
round:
  - phase: Prepare
    proposals: [1, 2]
  - phase: Accept
    proposal: 3
    value: "update"
```

**Flow:**

1. NodeA initiates transaction
2. Read Phase
3. NodeB and NodeC respondwith lock state
4. Consensus Phase
5. Leader proposes update
6. Following nodes vote forproposal
7. Proposal accepted by majority
8. NodeA updates rows

