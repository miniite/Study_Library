---
type: "practice"
difficulty: "Deep"
---

# A read replica is 8 seconds behind the primary and users report reading stale data right after writing. How would you offer read-your-own-writes without forcing all reads to the primary?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Primary-replica Databases](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Clickstream_Counting_Pipeline/Primary-replica_Databases.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve the issue of read replicas being 8 seconds behind the primary, leading to stale data, while still providing "read your own writes" functionality, we need a robust system design that ensures consistency and reliability. This problem is common in distributed database systems where read replicas are used for scaling reads.

The key idea here is to leverage optimistic concurrency control and implement a strategy that allows reads from any replica but ensures that the latest write is committed on the primary before allowing it back into the system. Let's break this down step by step:

Understand the Problem: Users report reading stale data right after writing, which indicates a race condition where reads are happening too soon after writes.
Define the Goal: Ensure that a client can read their own write without forcing all reads to the primary database, while maintaining strong consistency for this specific use case.

#### Assumptions
We have multiple read replicas.
Writes always go through the primary.
The system must handle high traffic and scale efficiently.

To achieve this, we can implement a combination of techniques including optimistic concurrency control, timestamp-based validation, and eventual consistency checks:

Step-by-Step Solution
Write to Primary: When a write operation is initiated by a client, it first sends the write request to the primary database.
Acknowledge Write on Primary: The primary acknowledges the write operation once it has been successfully written.
Immediate Read from Any Replica (Optional): To provide a fast response to the client, an optional initial read can be performed from any replica. However, this read should not be treated as authoritative until the primary confirms the write.
Primary Confirms Write: Once the primary has committed the write operation and it is safe to reflect the change in all replicas, it will broadcast a confirmation message to all replicas (including the ones that acknowledged the initial read).
Replica Validation on Read: When a client performs a read from any replica, the system checks if the data is up-to-date by comparing the timestamp of the latest committed write with the last write time of the client's operation.
Retry Mechanism: If the read returns stale data (due to replication lag), the client should be directed back to the primary for a definitive read.

#### Implementation
Strategy
Timestamp-Based Validation:
Each write transaction is timestamped and associated with a unique identifier.
Reads include a check of this timestamp against the latest committed write on the replica.
Eventual Consistency Check:
Implement a mechanism to ensure that reads from replicas eventually reflect the latest state, but allow for some latency.

#### Example
Workflow
Write Operation (Client A):
PLAINTEXT
ClientA -> Write Request: {timestamp: 100, data: "update"}
Primary -> Acknowledge Write: {timestamp: 100}
Immediate Read from Replica B (Optional):
PLAINTEXT
ClientA -> Read Request to ReplicaB
Primary Broadcasts Commit:
PLAINTEXT
Primary -> Confirm Write: {timestamp: 100, identifier: "write_id_1"}
Replica Validation on Read (Client A):
PLAINTEXT
ClientA -> Resubmit Read to ReplicaB with timestamp check
ReplicaB -> Check Timestamp against Commit (fails)
ClientA -> Redirected to Primary for Definitive Read

#### Closing Summary
and Common Mistakes

By implementing a hybrid approach of optimistic concurrency control and timestamp validation, we can provide read-your-own-writes functionality while minimizing the impact of replication lag. Key challenges include ensuring that the system handles edge cases where clients might be stuck in retries or dealing with inconsistent data states.

#### Common mistakes
include not properly handling edge cases such as:

Stale Replicas: Not checking the latest commit timestamp on each read.
Retry Loops: Clients being caught in infinite retry loops if the primary is unavailable.
Replication Lag: Clients reading from a replica that is still lagging behind.

#### Follow-Up
Questions
How would you handle scenarios where the primary is down?
What mechanisms can be implemented to minimize the impact of network partitions?
Can this approach be extended to other types of consistency models (e.g., eventual consistency)?

**Comparison:**

| Step | Action                  | Input                | Output              |
|------|-------------------------|----------------------|--------------------|
| 1    | Write to Primary        | {timestamp: 100, data: "update"}   | Acknowledge Write: {timestamp: 100}   |
| 2    | Immediate Read from B   |                      | Check Timestamp     |
| 3    | Validate on Replica B   | Fail                 | Redirect to Primary |

**Code:**

```python
def write_to_primary(data):
    # Send write request to primary and get an acknowledgment timestamp
    return {"timestamp": 100, "identifier": "write_id_1"}

def read_from_replica(timestamp):
    # Check if the replica's last commit matches or exceeds the client's timestamp
    latest_commit = get_latest_commit_timestamp()  # Assume a function that retrieves this information
    if latest_commit >= timestamp:
        return True
    else:
        redirect_to_primary()

def redirect_to_primary():
    # Redirect read operations to the primary for definitive data
```

**Example input / output:**

```text
Input (Client A -> Write):
- ClientA -> Write Request: {timestamp: 100, data: "update"}

Output (Primary Acknowledges Write):
- Primary -> Acknowledge Write: {timestamp: 100}

Input (Client A -> Immediate Read from Replica B):
- ClientA -> Read Request to ReplicaB

Output (Replica Validation Fails):
- ReplicaB -> Check Timestamp against Commit (fails)
- ClientA -> Redirected to Primary for Definitive Read
```

**Data:**

```json
{
  "write_request": {
    "timestamp": 100,
    "data": "update"
  },
  "primary_confirmation": {
    "timestamp": 100,
    "identifier": "write_id_1"
  }
}
```

**Flow:**

1. Fails
2. True
3. Write Request
4. Primary Acknowledges
5. Immediate Read from AnyReplica
6. Replica Validation
7. Redirect to Primary
8. Primary Confirms Write
9. Final Data Check
10. Definitive Read from Primary

