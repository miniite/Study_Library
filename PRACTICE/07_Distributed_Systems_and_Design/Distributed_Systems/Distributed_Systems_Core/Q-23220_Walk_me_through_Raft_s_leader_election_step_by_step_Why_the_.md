---
type: "practice"
difficulty: "Deep"
---

# Walk me through Raft's leader election step by step. Why the randomized election timeout, and what would go wrong if every follower used the same fixed timeout?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Leader Election](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Idempotent_Ingestion/Leader_Election.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Raft is a consensus algorithm designed to manage leader elections in distributed systems. The core of Raft's leader election mechanism involves a series of steps where followers periodically check their current state and potentially switch to become leaders if they meet certain conditions. The randomized election timeout is a critical component that ensures the leader election process is robust against network partitions and prevents oscillations.

To begin, let’s break down the key aspects of Raft's leader election algorithm:

States: Nodes can be in one of three states: Follower, Candidate, or Leader.
Election Timeout: This is a random interval between 150ms and 300ms, chosen by each follower to ensure that the system remains robust against network partitions.
Voting Process: When a node transitions from Follower to Candidate, it broadcasts its request for vote.

Now, let’s walk through Raft's leader election step by step:

Current State Check:

Each follower checks its current state every few milliseconds.

Election Initiation:

If a follower becomes a candidate, it sets an election timeout and sends a request for vote to other nodes.

Vote Request Process:

A candidate node sends a RequestVote message to all other nodes, including itself.
The response includes the current term number and the candidate ID.

Vote Granting:

A follower that receives a RequestVote request will respond with its current term number and decide whether to grant or deny the vote based on conditions like having seen a higher-numbered log entry from another leader.

Leader Election Outcome:

If a candidate node gets more than half of the votes, it becomes the leader.
Once elected as leader, the node transitions to Leader state and starts sending heartbeat messages to maintain leadership.

The randomized election timeout is crucial because:

Avoiding Oscillations: With fixed timeouts, nodes could fall into a deadlock if they all try to become leaders at the same time due to network delays.
Partition Resilience: Randomized timeouts help ensure that even in the presence of network partitions, nodes will eventually converge on a single leader.

If every follower used the same fixed timeout, several issues would arise:

Synchronous Leader Election: All followers might simultaneously transition to Candidate state due to a global network delay.
Oscillations: Continuous leader elections could lead to frequent leadership changes, which can cause instability and unnecessary work for all nodes.

Here’s a step-by-step illustration of the process:

Step 1: Follower Checks State
Assume node A is currently in Follower state.
Node A checks its current term number and state every few milliseconds.
Step 2: Election Initiation (Candidate)
If node A detects that it has not received a heartbeat from the leader for several rounds, it transitions to Candidate.
Node A sets its election timeout between 150ms and 300ms.
Node A sends RequestVote messages to all nodes.
Step 3: Vote Request Process
Node B receives a RequestVote message from node A. Node B checks its own term number, current state, and log entries.
If conditions are met (e.g., no higher-term leader), node B grants the vote to node A.
Node B sends a response indicating that it voted for node A.
Step 4: Leader Election Outcome
Suppose nodes C, D, and E grant votes to node A. Node A now has more than half of the votes.
Node A transitions to Leader state and starts sending heartbeat messages.
All other nodes recognize node A as leader and transition to Follower.
Step 5: Leader Sends Heartbeats
The new leader sends periodic heartbeats (e.g., AppendEntries requests) to ensure its leadership status.

This process ensures that the system remains robust against network partitions, reduces oscillations, and elects a single leader efficiently.

**Comparison:**

```
State	Action
Follower	Checks current term, transitions to Candidate on timeout
Candidate	Sets election timeout, sends RequestVote requests
Leader	Sends heartbeat messages
```

**Code:**

```python
# Example of sending RequestVote message in Python
def send_request_vote(node_id, term):
    # Send RequestVote request to other nodes
    for follower in followers:
        response = follower.send_request_vote(term)
        if response.granted_vote:
            increment_votes_count()

    if votes_count > (len(followers) // 2 + 1):
        transition_to_leader()

# Example of transitioning to Leader state
def transition_to_leader():
    print("Node is now a leader.")
```

**Example input / output:**

```text
Input:
- Node A transitions to Candidate state.
- Sets election timeout between 150ms and 300ms.

Output:
- Node B, C, D, E send vote responses.
- Node A receives more than half of the votes.

Dry-run Response:
Node A becomes leader and starts sending heartbeats.
```

**Data:**

```json
{
  "followers": ["A", "B", "C", "D", "E"],
  "term": 1,
  "heartbeat_interval": 300ms,
  "election_timeout_min": 150ms,
  "election_timeout_max": 300ms
}
```

**Flow:**

1. Yes
2. Yes
3. Current Term Check
4. Is Leader Heartbeat Timeout?
5. Follower Becomes Candidate
6. Election Timeout RandomizedBetween 150ms-300ms
7. Sets Election Timeout, SendsRequestVote
8. Receive More than Half Votes?
9. Transition to Leader
10. Send Heartbeat Messages

