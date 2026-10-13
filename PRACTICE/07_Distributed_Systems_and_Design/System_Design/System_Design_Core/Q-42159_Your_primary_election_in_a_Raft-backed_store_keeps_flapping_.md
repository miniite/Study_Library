---
type: "practice"
difficulty: "Expert"
---

# Your primary election in a Raft-backed store keeps flapping under network jitter. How does Raft pick a leader, why do split-votes happen, and how would you tune election timeouts?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Primary-replica Databases](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Clickstream_Counting_Pipeline/Primary-replica_Databases.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview

Raft is a consensus algorithm used to coordinate nodes in a distributed system. In your scenario, the primary election process in Raft is failing due to network jitter, causing frequent elections and potentially leading to a flapping leader. Understanding how Raft picks leaders, why split votes happen, and tuning election timeouts are crucial for stabilizing the system.

#### Triaging the Question
Understanding Raft's Leader Election Process:
Raft nodes compete for leadership by sending heartbeat messages.
A node that receives a majority of votes from other nodes becomes the leader.
Split Votes and Their Causes:
Split votes occur when nodes cannot agree on a single leader due to network delays or misconfiguration.
Tuning Election Timeouts:
Election timeouts are critical in determining how long a node waits before initiating an election.
Detailed Solution from First Principles:
How Raft Picks a Leader

Heartbeats and Elections:

Nodes continuously send heartbeats to the network.
If a node does not receive a heartbeat from the current leader within its heartbeat interval, it assumes the leader has failed and starts an election process.

Election Process:

A candidate node increases its term number and votes for itself.
The node sends a request vote message to other nodes with its term number and vote details.
If a majority of nodes agree (vote), the candidate becomes the leader.
Why Split Votes Happen
Network Jitter:
Network delays can cause messages to be received out of order, leading to nodes receiving different terms from each other.
Message Loss or Duplication:
Lost or duplicated messages can result in some nodes thinking a node has failed and initiating an election, while others continue with the current leader.
Tuning Election Timeouts

Understanding Timeout Values:

The election timeout is a critical setting that determines how long a node waits before starting an election.
It is typically set as a random value between two fixed points to avoid nodes syncing too closely and reducing the chances of split votes.

Adjusting Timeout Settings:

Shorter Timeouts: Can reduce the frequency of elections but may increase the likelihood of network jitter causing split votes.
Longer Timeouts: Reduce the chance of split votes due to network issues but can lead to longer periods without leader activity, which might not be ideal for some applications.

Practical Tuning:

A common practice is setting election_timeout as a random value between 150ms and 300ms.
This range provides a balance between reducing the frequency of elections and minimizing the impact of network jitter.

#### Closing Summary

By understanding Raft's leader election process, recognizing how split votes occur due to network issues, and tuning election timeouts appropriately, you can significantly improve the stability of your system. The key is finding a balance that minimizes the impact of network jitter without compromising the reliability of the consensus algorithm.

#### Common Mistakes
and Follow-Ups
Ignoring Network Jitter: Failing to account for network delays can lead to frequent elections.
Setting Too Short Timeouts: Can cause too many unnecessary elections, leading to performance issues.
Not Considering Node Distribution: In a highly distributed system, timeouts might need to be adjusted based on the average network latency.

**Comparison:**

```
Node	Network Latency (ms)	Election Timeout (ms)
1	20	178
2	30	265
3	40	194
4	10	245
5	25	287
```

**Code:**

```python
# Example of setting election timeout in Python (simplified)
import random

ELECTION_TIMEOUT_MIN = 150
ELECTION_TIMEOUT_MAX = 300

def set_election_timeout():
    return random.randint(ELECTION_TIMEOUT_MIN, ELECTION_TIMEOUT_MAX) * 1000  # Convert to ms

election_timeout = set_election_timeout()
print(f"Set election timeout to {election_timeout}ms")
```

**Example input / output:**

```text
Input:
- Node count: 5
- Network latency: [20ms, 30ms, 40ms, 10ms, 25ms]
- Election timeout range: 150ms to 300ms

Output:
Election timeouts for each node will be set as follows (in ms):
Node 1: 178
Node 2: 265
Node 3: 194
Node 4: 245
Node 5: 287
```

**Data:**

```json
{
  "node_count": 5,
  "network_latency_ms": [20, 30, 40, 10, 25],
  "election_timeout_min": 150,
  "election_timeout_max": 300
}
```

**Flow:**

1. No
2. Yes
3. No
4. Detect Node Failure
5. Start Election
6. Is Current Leader?
7. Send Request Vote Message toOther Nodes
8. Majority of Votes?
9. Become Leader
10. Start Heartbeat Process
11. Continue as Follower

