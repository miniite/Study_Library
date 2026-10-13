---
type: "practice"
difficulty: "Deep"
---

# Explain quorum reads and writes in a Dynamo-style store: with N=3, when does R+W>N guarantee you read your write, and what breaks that guarantee during a partition?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Primary-replica Databases](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Clickstream_Counting_Pipeline/Primary-replica_Databases.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a Dynamo-style distributed database, quorum reads and writes are fundamental to ensuring data consistency across multiple nodes while providing high availability and fault tolerance. Specifically, with (N = 3) (replicas), (R + W > N) guarantees that at least one replica will have the most recent write when read or written. However, this guarantee can be broken under certain conditions, particularly during a network partition.

#### Triage
The question asks for an explanation of how quorum reads and writes work in a Dynamo-style store with three replicas ((N = 3)), focusing on the condition (R + W > N) and what circumstances might violate this guarantee. This involves understanding the consistency model, the role of read and write operations, and the impact of network partitions.

#### Assumptions

We are discussing a consistent read and write operation.
(R + W > 3), meaning both reads and writes require a majority of replicas to be acknowledged.
The system uses token ring routing for consistent hashing and handles failures by reassigning tokens.

#### First-principles solution
In a Dynamo-style store with three replicas, the quorum read and write model ensures that at least one replica will have the latest data after a write. This is achieved by setting (R + W > N), where typically both reads and writes require 2 out of 3 replicas to confirm.

For a write operation:

(W = 2): Two replicas must acknowledge the write.
For a read operation:
(R = 1): One replica must be queried for the data.
(R + W > N \Rightarrow 1 + 2 > 3) is not satisfied, so no guarantee of reading the most recent write.

However, if we set both (R = 2) and (W = 2):

(R + W = 4 > 3) (the number of replicas), ensuring at least one replica will have the latest data when read.

Now, let’s consider what breaks this guarantee during a partition:

Network Partition: If a network partition occurs such that some nodes are isolated from others, the quorum required for writes may not be achievable.
Token Ring Routing Failure: If the token ring routing fails, it can lead to inconsistent state across replicas.

For example, if a write is issued and two of the three replicas are in an isolated network segment due to a partition:

The third replica acknowledges the write.
A read operation from any one of these replicas will not guarantee reading the most recent write because only one replica has updated data, while the others might still have stale information.

#### Closing summary
In summary, (R + W > N) in a Dynamo-style store with three replicas ensures that at least one replica will have the latest data when both reads and writes require 2 out of 3 confirmations. However, network partitions or routing failures can break this guarantee by isolating nodes such that not all required quorums can be satisfied.

#### Common mistakes
and follow-ups:

Misunderstanding the requirement for (R + W > N).
Failing to consider the impact of network partitions on the system’s ability to achieve quorum.
Not recognizing that token ring routing is crucial in maintaining consistency across replicas.

**Comparison:**

| Operation | Replicas Required |
|-----------|------------------|
| Write     | R + W > N         |
| Read      | R = 2             |

**Code:**

```python
# Example code to illustrate token ring routing and replica updates
class DynamoNode:
    def __init__(self, node_id):
        self.node_id = node_id
        self.tokens = set()

    def add_token(self, start_token, end_token):
        self.tokens.update(range(start_token, end_token))

def assign_tokens(nodes):
    # Simulate token assignment in a ring
    nodes[0].add_token(1, 5)
    nodes[1].add_token(5, 9)
    nodes[2].add_token(9, 13)

# Example usage
nodes = [DynamoNode(i) for i in range(3)]
assign_tokens(nodes)
```

**Example input / output:**

```text
Input:
- Write operation with R = 2 and W = 2

Output:
- If a partition occurs isolating two nodes, the third node might have the latest write.
- Read from any of the isolated nodes will return stale data.

Example dry-run:
1. Write operation: Node 0 -> success (replicas [1, 2])
2. Read operation on Node 1: returns old data
```

**Data:**

```json
{
  "replicas": [
    {"node_id": 0, "tokens": [1, 2, 3]},
    {"node_id": 1, "tokens": [4, 5, 6]},
    {"node_id": 2, "tokens": [7, 8, 9]}
  ]
}
```

**Flow:**

1. Write Operation
2. Replica 0 Acknowledges
3. Replica 1 and 2 Acknowledge
4. Write Successful
5. Read from Replica 1
6. Old Data Returned
7. Network Partition Occurs
8. Node 0 and Node 2 Isolated
9. Node 1 Acknowledges Write
10. Read from Node 1
11. Most Recent Data Returned

