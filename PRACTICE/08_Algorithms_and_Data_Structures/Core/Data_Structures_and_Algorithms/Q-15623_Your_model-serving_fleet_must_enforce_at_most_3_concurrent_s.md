---
type: "practice"
difficulty: "Deep"
---

# Your model-serving fleet must enforce 'at most 3 concurrent sessions per user' across nodes; where does an in-memory counter break under node failure and what distributed structure keeps it correct?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Concurrent Counters and CAS](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Concurrent_Counters_and_CAS.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a system to enforce "at most 3 concurrent sessions per user" using a distributed architecture, we need to ensure that the state (in this case, session counts) is consistent across multiple nodes in the event of node failures. An in-memory counter can break under such scenarios, as it does not inherently provide fault tolerance or consistency guarantees. To address this, we need to use a more robust data structure, such as a distributed hash table (DHT).

The question essentially asks where an in-memory counter breaks down and how a distributed structure can keep the state correct even if nodes fail.

#### Triaging the Question
Understanding the Requirement: We need to ensure that each user has at most 3 concurrent sessions across all nodes.
What is Being Probed:
Understanding of in-memory vs. distributed data structures.
Knowledge of fault tolerance and consistency models.
Ability to reason about edge cases, particularly node failures.

#### Assumptions
We are dealing with a multi-node system where each node can serve requests independently.
User sessions need to be tracked globally across all nodes.
Node failures may occur at any time.

#### First Principles Solution
In-Memory Counter Breakdown

An in-memory counter is stored on a single node. If that node fails, the session count for users hosted on that node will be lost. This can cause issues such as:

Incorrect Session Counts: The global state of sessions for affected users becomes incorrect.
Session Overflows: Users might exceed their allowed number of concurrent sessions if not properly managed.
Distributed Data Structure: Count-Min Sketch (CMS)

A better solution is to use a distributed structure like the Count-Min Sketch, which can tolerate node failures and still provide an approximate count of user sessions across nodes.

Step-by-Step Solution

Distributed Hash Table (DHT):

Use a DHT to distribute the session counts across multiple nodes.
Each node maintains its portion of the state but collaborates with other nodes for consistency.

Count-Min Sketch:

Implement Count-Min Sketch on each node.
When a new session is created, increment the sketch for the user's ID hash value.
Periodically sync sketches between nodes to ensure consistency and handle failures.

Node Failure Handling:

If a node fails, its state can be reconstructed by other live nodes through periodic syncs or replication mechanisms.
Use techniques like Paxos or Raft for distributed consensus to manage state updates reliably.

Load Balancing and Synchronization:

Ensure that load balancing is done in a way that minimizes the likelihood of having too many concurrent sessions for any user.
Implement synchronization protocols (e.g., epochs, timestamp-based updates) to handle race conditions during session creation and deletion.

#### Closing Summary

By using a distributed structure like Count-Min Sketch or a DHT with periodic syncs, we can ensure that each user has at most 3 concurrent sessions globally across nodes. The key idea is to distribute the state (session counts) such that it can be consistent even if individual nodes fail.

#### Common Mistakes
and Follow-Ups
Not Considering Node Failures: Failing to acknowledge the possibility of node failures and their impact on global state.
Ignoring Consistency Models: Assuming in-memory counters will always provide the correct count without any failures.
Insufficient Synchronization: Not implementing proper synchronization mechanisms to handle concurrent updates.

#### Follow-ups
could include discussing specific consistency models (e.g., CAP theorem implications), trade-offs between different distributed algorithms, and how to optimize for performance while maintaining correctness.

**Comparison:**

```
Node	Hash Function 1	Hash Function 2
A	hash1: 2	hash2: 0
B	hash1: 1	hash2: 3
```

**Code:**

```python
# Example of initializing a Count-Min Sketch in Python
from collections import defaultdict

class CountMinSketch:
    def __init__(self, w, d):
        self.w = w  # Number of hash functions
        self.d = d  # Array size
        self.sketches = [defaultdict(int) for _ in range(self.w)]

    def update(self, user_id, value=1):
        for i in range(self.w):
            h = hash(user_id + str(i)) % self.d
            self.sketches[i][h] += value

    def query(self, user_id):
        min_count = float('inf')
        for i in range(self.w):
            h = hash(user_id + str(i)) % self.d
            count = self.sketches[i].get(h, 0)
            if count < min_count:
                min_count = count
        return min_count

# Example usage
cms = CountMinSketch(w=3, d=1000)
cms.update('user123', value=1)
print(cms.query('user123'))  # Output: 1
```

**Example input / output:**

```text
Input:
- User 'user123' creates a new session.

Output:
- NodeA and NodeB update their Count-Min Sketches.
- Syncs occur periodically to maintain consistency across nodes.
```

**Data:**

```json
{
  "users": [
    {"id": "user123", "sessions": ["session1", "session2"]},
    {"id": "user456", "sessions": ["session3"]}
  ],
  "nodes": [
    {"node_id": "nodeA", "sketch": {"hash1": 2, "hash2": 0}},
    {"node_id": "nodeB", "sketch": {"hash1": 1, "hash2": 3}}
  ]
}
```

**Flow:**

1. User creates session
2. Node updates sketch
3. Sketches sync periodically
4. Failures handled through syncs

