---
type: "practice"
difficulty: "Deep"
---

# You want O(1) lookup of 'the current leader for this shard' but leadership changes on failover; what versioned map or epoch structure prevents a stale reader from routing to a demoted node?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Concurrent Counters and CAS](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Concurrent_Counters_and_CAS.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve the problem of maintaining "the current leader" in a distributed system while ensuring O(1) lookup and handling failovers, we need a data structure that supports efficient retrieval of the most recent leader version and allows us to track the history of leadership changes. The key challenge is preventing a stale reader from routing to a demoted node after a failover.

#### Triage

The question is essentially about designing an efficient and robust system for managing the current leader in a distributed context, where leaders can change due to failovers. We need to ensure that lookups are fast (O(1)) and that historical leadership information is preserved without causing staleness issues.

#### First-Principles Solution

To achieve this, we will design an epoch-based versioned map data structure. Each leader update increments the epoch, and nodes store their status with the corresponding epoch. This way, a reader can quickly determine the current leader by checking the latest epoch and verify if it has not been demoted.

Step-by-Step Solution
Epoch Management: Maintain an incrementing epoch counter that increases every time the leadership changes.
Leader Tracking: Store each node's status (leader or follower) with the corresponding epoch in a map.
O(1) Lookup: Use a hash table or other O(1) data structure to store nodes and their statuses by epoch.
Staleness Prevention: Ensure that only the latest epoch leader is considered valid, preventing readers from using outdated information.

#### Implementation

We will use a hash map where keys are node identifiers and values are structs containing the status (leader/follower) and the epoch of the last update.

PYTHON
class LeadershipMap:
    def __init__(self):
        self.leadership_map = {}  # Stores nodes with their status by epoch

    def update_leader(self, node_id: str, is_leader: bool, current_epoch: int):
        if node_id in self.leadership_map and self.leadership_map[node_id]['epoch'] >= current_epoch:
            return
        self.leadership_map[node_id] = {'is_leader': is_leader, 'epoch': current_epoch}

    def get_current_leader(self) -> str:
        max_epoch = -1
        current_leader = None
        for node_id, status in self.leadership_map.items():
            if status['epoch'] > max_epoch:
                max_epoch = status['epoch']
                current_leader = node_id
        return current_leader

    def is_node_leader(self, node_id: str) -> bool:
        if node_id not in self.leadership_map:
            return False
        status = self.leadership_map[node_id]
        return status['is_leader'] and status['epoch'] == max([s['epoch'] for s in self.leadership_map.values()])

#### Closing Summary

This approach ensures that the current leader can be determined efficiently (O(1)) while maintaining historical leadership information. By using epochs, we prevent readers from routing to demoted nodes, ensuring consistency and correctness.

#### Common Mistakes/Follow-Ups
Staleness Issues: Ensure that all nodes update their status with the correct epoch.
Concurrency: Handle race conditions when multiple updates are happening simultaneously.
Performance Overhead: Minimize overhead by optimizing epoch increments and status checks.

