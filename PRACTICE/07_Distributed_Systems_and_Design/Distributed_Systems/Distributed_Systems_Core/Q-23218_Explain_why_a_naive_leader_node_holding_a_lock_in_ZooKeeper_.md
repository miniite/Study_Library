---
type: "practice"
difficulty: "Expert"
---

# Explain why a naive 'leader = node holding a lock in ZooKeeper' design can still allow two nodes to believe they're leader. What is a fencing token and where exactly must it be checked?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Distributed Locks and Fencing](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Idempotent_Ingestion/Distributed_Locks_and_Fencing.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In distributed systems, particularly when using Apache ZooKeeper for leader election, a naive approach of simply having a node hold a lock can lead to situations where two nodes incorrectly believe they are the leader. This is due to potential inconsistencies in lease management and network partitions. The core issue arises from the lack of robust mechanisms to handle node failures or stale information effectively.

To address this problem, the concept of a "fencing token" is introduced. A fencing token serves as a mechanism to prevent multiple nodes from incorrectly claiming leadership when they have lost contact with their peers but still believe they are leaders. The key idea is that every write operation should be accompanied by a unique identifier (the fencing token) which must be checked before the node considers itself the leader.

#### Triage

The question probes your understanding of distributed consensus, leader election mechanisms in ZooKeeper, and the importance of robust failure detection and recovery strategies. We'll assume ZooKeeper is being used for its inherent distributed nature and reliability guarantees. Common assumptions include a network with intermittent failures and nodes that can lose contact with others.

Step-by-Step Solution
1. Understanding the Naive Leader Election Design

In a naive design, each node checks if it holds a lock or Znode in ZooKeeper to determine leadership. However, this approach fails when nodes go down and come back up during a network partition. The leader might still think it’s leading because its state was not updated due to the network split.

2. Introduction of Fencing Tokens

A fencing token is a unique identifier associated with each transaction or operation in ZooKeeper. When a node performs an operation, it includes this token. Before claiming leadership, nodes must check if their current token matches any existing tokens in ZooKeeper. If not, the node should not consider itself as the leader.

3. Implementation Details
Token Generation: Each write or update operation generates a unique fencing token.
Token Check: Every node before updating its leadership state checks if its token is the most recent one stored in ZooKeeper.

#### Cons
istency Checks: Nodes periodically sync their tokens with each other to ensure consistency and prevent stale leadership claims.
Full Solution

When two nodes go down simultaneously during a network partition, they might both try to reestablish leadership when the network recovers. To avoid this, upon recovering, each node checks its fencing token against the most recent one in ZooKeeper. If it finds that another node has a more recent token, it will gracefully step down instead of claiming leadership.

#### Closing Summary
and Common Mistakes/Follow-Ups

The use of fencing tokens ensures that only one node can claim leadership at any given time, even during network partitions or node failures. This prevents the "split-brain" scenario where multiple nodes incorrectly believe they are leaders. Key mistakes include forgetting to update tokens regularly, missing the token check in critical operations, and not handling edge cases like network delays.

#### Follow-ups
could involve discussing how fencing tokens interact with other ZooKeeper features (e.g., session timeouts) or exploring alternative leader election strategies like Raft or Paxos.

**Comparison:**

```
Node	Fencing Token
A	token12345
B	token67890
C	token98765

Node C has the most recent fencing token and should be considered as the leader.
```

**Code:**

```python
# Example pseudo-code for checking a fencing token in ZooKeeper

import zookeeper  # Hypothetical library

def update_fencing_token(node_id, operation):
    token = generate_unique_token()
    znode_path = f"/fencing_tokens/{node_id}"

    result = zookeeper.setData(znode_path, token.encode(), version=-1)
    if not result:
        print("Failed to set fencing token")
    return token

def check_fencing_token(node_id):
    znode_path = f"/fencing_tokens/{node_id}"
    current_token = zookeeper.getData(znode_path)[0].decode()

    # Simulate token comparison
    if current_token != expected_token:
        print("Node is not the leader")
    else:
        print("Node is the leader")
```

**Example input / output:**

```text
Input:
- Node A and B go down simultaneously during a network partition.
- Network recovers, both nodes try to re-establish leadership.

Output:
Node A and B check their fencing tokens against the most recent one in ZooKeeper. If they find that another node has a more recent token, they will not claim leadership but instead step down or continue as followers.
```

**Data:**

```yaml
# Example fencing tokens schema for ZooKeeper nodes

fencing_tokens:
  - node_id: "node1"
    token: "token1234567890abcdefg"
  - node_id: "node2"
    token: "token9876543210fedcba"
```

**Flow:**

1. True
2. False
3. Node goes down
4. Network recovers
5. Check current token againstZooKeeper
6. Token matches
7. Step down or continuefollower
8. Token does not match

