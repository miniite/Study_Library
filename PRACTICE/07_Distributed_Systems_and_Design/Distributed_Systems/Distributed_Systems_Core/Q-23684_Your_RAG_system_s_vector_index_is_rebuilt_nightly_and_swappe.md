---
type: "practice"
difficulty: "Deep"
---

# Your RAG system's vector index is rebuilt nightly and swapped in. Explain how you'd make the index swap atomic across a distributed retrieval fleet so queries never hit a half-built index, and how you roll back a bad index build.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Keeping Indexes in Sync via CDC](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Clock_Skew_and_Time_Windows/Keeping_Indexes_in_Sync_via_CDC.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that the vector index swap is atomic in a RAG system's distributed retrieval fleet, we need to guarantee zero downtime for queries by swapping out an old index only after the new one has fully been built and verified. Additionally, if a bad build occurs, we must have robust rollback mechanisms to revert back to the previous stable version of the index. This requires careful design around state management, coordination between nodes, and efficient verification processes.

#### Triage

The core question is about designing an atomic swap mechanism for vector indices in a distributed system. The interviewer is probing how you handle seamless transitions without data loss or query disruptions. Key assumptions include:

Index Rebuild Frequency: Indexes are rebuilt nightly.
Distributed Retrieval Fleet: Nodes can be added or removed, and the fleet operates across multiple locations.
Rollback Mechanism: In case of a bad build, we need to quickly revert to the previous working version.

#### Solution
Step-by-Step Approach

Index Versioning:

Each index should have a unique version identifier. For instance, v1, v2, etc.

Preparation Phase:

The nightly rebuild starts by creating an initial empty state for the new version (e.g., v2).
Data is incrementally added to this new index until it reaches a stable state.
A health check mechanism ensures that the new index has met all quality standards before marking it as ready.

Swapping Mechanism:

When the new index passes validation, it becomes available for read-only access by a small subset of nodes.
The v2 index is gradually rolled out to more and more nodes over time, ensuring that each node updates its state independently but atomically through coordination.

Atomic Swap Execution:

Once the new index reaches sufficient coverage across the fleet (e.g., 90% of nodes), a global atomic swap command is issued.
This command ensures all nodes switch from v1 to v2, or if something goes wrong, they revert back.

Rollback Mechanism:

If any node detects an issue during the transition (e.g., degraded performance, anomalies), it signals a rollback signal.
The system coordinates a global rollback command that reverts all nodes back to v1 and initiates a fresh rebuild if necessary.

#### Closing Summary

The key idea is to use versioning, incremental updates, health checks, atomic switches, and robust rollback mechanisms to ensure seamless index transitions in a distributed retrieval fleet. This approach minimizes downtime and ensures data integrity even during complex operations like nightly rebuilds.

#### Common Mistakes/Follow-Ups

Mistake: Ignoring the state synchronization between nodes.

#### Follow-Up
What happens if network partitions prevent nodes from updating simultaneously? Discuss how to handle partial failures and ensure eventual consistency.

Mistake: Failing to implement a rollback mechanism.

#### Follow-Up
How do you detect issues during the index build process, and what are your strategies for handling them?

**Comparison:**

```
Node ID	IP Address	Current Index Version
1	192.168.1.1	v1
2	192.168.1.2	v1
...	...	...
Atomic Switch Lock Status	Current Version
Acquired	v1
```

**Code:**

```python
# Example Health Check Function
def check_index_health(index_version):
    # Implement checks like performance metrics, data integrity, etc.
    pass

# Example Atomic Swap Function
def perform_atomic_swap(current_version, new_version):
    if not check_index_health(new_version):
        return False  # Rollback and start fresh rebuild

    for node in retrieval_fleet:
        node.update_index_state(current_version, new_version)

    global atomic_switch_lock:
    with atomic_switch_lock:
        current_version = new_version
        print(f"Index version switched to {new_version}")
    return True

# Example Rollback Function
def perform_rollback(index_version):
    for node in retrieval_fleet:
        node.update_index_state(new_version, index_version)

    global atomic_switch_lock:
    with atomic_switch_lock:
        current_version = index_version
        print(f"Rolling back to index version {index_version}")
```

**Example input / output:**

```text
Input:
- `perform_atomic_swap("v1", "v2")`

Output:
Node 1: Updating index state from v1 to v2
Node 2: Updating index state from v1 to v2
...
Index version switched to v2

Rollback Example:
Input:
- `perform_rollback("v2")`

Output:
Node 1: Rolling back index state from v2 to v1
Node 2: Rolling back index state from v2 to v1
...
Rolling back to index version v1
```

**Data:**

```json
{
  "retrieval_fleet": [
    {"id": 1, "ip": "192.168.1.1", "state": "v1"},
    {"id": 2, "ip": "192.168.1.2", "state": "v1"},
    ...
  ],
  "current_version": "v1",
  "new_version": "v2"
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Is New Index Ready?
5. Mark New Index As Available
6. Perform Global Atomic Swap
7. All Nodes Switch To New Index
8. End
9. Health Check Failed - Rollback
10. Initiate Fresh Rebuild

