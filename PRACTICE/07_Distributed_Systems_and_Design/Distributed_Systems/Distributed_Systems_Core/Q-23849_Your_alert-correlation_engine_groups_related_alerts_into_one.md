---
type: "practice"
difficulty: "Deep"
---

# Your alert-correlation engine groups related alerts into one incident, but a network partition split its two nodes and both opened a duplicate incident for the same outage. Explain how the partition caused the double-fire and how you'd design leadership so only one node correlates a given alert stream.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Gossip and Deduplication](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Clock_Skew_and_Time_Windows/Gossip_and_Deduplication.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In distributed systems, ensuring high availability often involves replicating critical components across multiple nodes to prevent single points of failure. However, network partitions can disrupt this balance and lead to unexpected behaviors, such as duplicated incidents in an alert-correlation engine. In this scenario, a network partition split the two nodes responsible for processing alerts into separate regions, each opening its own incident for the same outage.

#### Triage

The core issue is that the alert-correlation logic needs to be designed to handle network partitions gracefully. Specifically, we need to ensure that only one node processes and correlates an alert stream at a time, preventing duplicate incidents when nodes are split by a partition. The solution involves implementing a distributed leadership mechanism (e.g., leader election or lease-based coordination) to manage which node is responsible for processing the alerts.

Step-by-Step Solution

Understanding the Problem:

A network partition occurred, splitting the alert-correlation engine into two separate regions.
Each node independently opened an incident for the same outage due to lack of coordination.

Leadership Mechanism:

Introduce a distributed leadership mechanism where only one node is the leader at any given time.
The leader is responsible for processing and correlating alerts, while other nodes remain in a follower state.

Leader Election:

Implement a leader election algorithm (e.g., Raft or Paxos) to elect the leader among the nodes.
Ensure that once elected, the leader maintains its leadership status until it steps down voluntarily or fails.

Communication and Coordination:

Design a communication protocol between leader and followers for coordination.
Use heartbeat messages from the leader to confirm its continued availability and validity.

Incident Correlation:

Implement logic in the leader node to correlate alerts based on common criteria (e.g., alert source, timestamp, event type).
Ensure that correlated incidents are not duplicated by maintaining a unique incident ID or correlation token.

Handling Partitions:

When a partition occurs and nodes are split, ensure that only one region retains leadership.
Use quorum-based decisions to determine which side of the partition should retain leadership.

Failover Strategy:

In case the current leader fails or steps down, quickly elect a new leader from the available nodes.
Ensure failover is quick and seamless to minimize downtime during network partitions.

#### Closing Summary

By implementing a distributed leadership mechanism with leader election and coordinated incident correlation, we can ensure that only one node processes and correlates alerts in any given partition. This approach prevents duplicate incidents and ensures high availability while maintaining the integrity of alert-correlation logic.

#### Common Mistakes/Follow-Ups
Mistake 1: Failing to implement a robust leader election algorithm.
Mistake 2: Not designing proper communication protocols between leader and followers.
Mistake 3: Overlooking the handling of network partitions, leading to inconsistent states across nodes.

#### Follow-up
questions might include:

How do you ensure that the elected leader remains stable during network disruptions?
What failover strategies would you use if the current leader fails before it can step down gracefully?

**Comparison:**

```
Node	Leadership Status
node1	Leader
node2	Follower
```

**Code:**

```python
# Example implementation of a simple heartbeat-based leadership mechanism

import threading
from time import sleep

class LeaderElection:
    def __init__(self, nodes):
        self.nodes = nodes
        self.leader = None
        self.lock = threading.Lock()

    def start_election(self):
        # Start the election process
        for node in self.nodes:
            if node == 'node1':
                self.elect_leader(node)

    def elect_leader(self, node):
        with self.lock:
            print(f"Elected {node} as leader")
            self.leader = node

    def heartbeat(self, node):
        while True:
            # Send heartbeat to maintain leadership
            if node == self.leader:
                print(f"Heartbeat from {node}")
            sleep(1)
```

**Example input / output:**

```text
Input:
- Node nodes = {"nodes": ["node1", "node2"], "leader": null, "heartbeatInterval": 1}

Output:
Elected node1 as leader
Heartbeat from node1
Heartbeat from node1
```

**Data:**

```json
{
  "nodes": ["node1", "node2"],
  "leader": null,
  "heartbeatInterval": 1
}
```

**Flow:**

1. Yes
2. No
3. Node 1
4. Is Leader?
5. Send Heartbeat
6. Stay as Leader
7. Follower
8. Wait for Leadership

