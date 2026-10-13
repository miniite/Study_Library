---
type: "practice"
difficulty: "Deep"
---

# Explain how a write-ahead log gives durability and enables replication and recovery. Walk through what happens on crash-recovery: redo vs undo, checkpoints, and why fsync matters.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Quorums and Consensus](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Quorums_and_Consensus.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand how a write-ahead log (WAL) provides durability in distributed systems, we need to break down its role in ensuring data consistency across multiple nodes and how it aids in crash recovery. The core idea is that WAL records all changes before they are applied, allowing the system to replay these logs after a failure or during recovery.

#### Triage
What the Question Is Asking

The question seeks an explanation of how WAL contributes to durability, replication, and recovery mechanisms. Specifically, we need to discuss:

Durability: Ensuring data is not lost in case of a crash.
Replication: How data is distributed across multiple nodes for fault tolerance.
Crash Recovery: Processes like redo (reapplying logs) and undo (rolling back changes).
Fsync: The importance of synchronizing writes to the disk.

#### First-Principles Solution

A WAL is a mechanism where every write operation is logged before being committed to stable storage. This ensures that all writes are recorded in the log first, making them durable even if the system crashes immediately after writing to memory but before committing to disk. Here’s how it works:

Write-Ahead Logging:

Each time a write operation occurs, the change is first written to the WAL.
Only once this record is successfully logged does the system proceed with committing the data to stable storage (e.g., a database file on disk).

Replication and Consistency:

In distributed systems, multiple nodes can replicate data by applying logs from the leader node or a primary node.
This ensures that all nodes have an up-to-date copy of the latest state.

Crash Recovery:

If the system crashes before committing to stable storage, it can recover by replaying the WAL entries to ensure consistency.
Two main recovery mechanisms are used: redo and undo.

Redo Operations:

During crash recovery, the system reads from the WAL and reapplies any operations that were only logged but not yet committed.
This ensures that all changes made before the failure are accounted for.

Undo Operations:

For some systems or certain types of writes (like deletions), an undo operation might be necessary to revert changes if they were partially applied and then a crash occurred.
However, this is less common and often managed by other mechanisms like transactional integrity.

Checkpoints:

Checkpoints are periodic snapshots that help in reducing the amount of WAL data needed to recover.
They allow the system to skip replaying parts of the log after a known good state.

Fsync:

Ensures that all writes to the WAL and stable storage are fully committed to disk.
It is crucial for ensuring durability, as it prevents data from being lost due to power failures or unexpected crashes.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, a write-ahead log ensures that all writes are recorded before they are applied to stable storage, providing a safety net against crashes. For distributed systems, this is crucial for durability and consistency. When the system fails, WAL helps in recovery through redo operations, ensuring data integrity. Checkpoints further optimize recovery processes.

#### Common mistakes
include not properly handling fsync calls, leading to data loss, or misunderstanding the importance of periodic checkpoints. Another pitfall is assuming that all changes need immediate undo operations, which can unnecessarily complicate systems and performance.

For follow-ups, one might explore advanced topics like log compactification in Kafka or the use of journaling in databases.

**Comparison:**

```
Operation	Time	Action
1	2023-10-01T14:00:00	Create
2	2023-10-01T14:05:00	Update
3	2023-10-01T14:10:00	Delete
Recovery Steps	Action
Checkpoint 1 (at 14:00)
Crash detected at 14:05	Redo operation 2
```

**Code:**

```python
# Example code to illustrate WAL usage (pseudo-code)
def write_to_log(entry):
    # Write entry to WAL first
    with open('wal_file', 'a') as wal:
        wal.write(f"{entry}\n")

    # Then attempt to commit the entry to stable storage
    try:
        db.commit_entry(entry)
    except Exception:
        # If commit fails, log failure and retry or handle recovery later
        pass

def crash_recovery():
    with open('wal_file', 'r') as wal:
        for line in wal.readlines():
            operation = parse_log_entry(line)
            if not is_committed(operation):
                apply_redo_operation(operation)
```

**Example input / output:**

```text
Input (before crash):
Write operation to create user 'user123' with value 'initial_value'
Output (after recovery, assuming crash occurred between first and second write):
1. Create user 'user123'
2. Update user 'user123' to 'new_value'
```

**Data:**

```json
{
  "wal_data": [
    {"time": "2023-10-01T14:00:00", "action": "create", "key": "user123"},
    {"time": "2023-10-01T14:05:00", "action": "update", "key": "user123", "value": "new_value"},
    {"time": "2023-10-01T14:10:00", "action": "delete", "key": "user123"}
  ]
}
```

**Flow:**

1. No
2. Yes
3. No
4. Log Entry
5. Is committed?
6. Redo Operation
7. Was checkpointed?
8. Skip to next log entry
9. Redo operation

