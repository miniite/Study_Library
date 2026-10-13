---
type: "practice"
difficulty: "Deep"
---

# Explain how an LSM handles the read-your-own-writes guarantee within a single session. Why is it trivially satisfied by checking the memtable first, and how does this change with a follower/replica read?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ LSM Write Path WAL and Compaction](<../../../../01_Databases_and_SQL/Database_Internals/Storage_Engines/Tiered_Storage_in_LSM_Engines/LSM_Write_Path_WAL_and_Compaction.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In an LSM (Log-Structured Merge) database, ensuring "read-your-own-writes" within a single session is crucial for consistency. This guarantee means that any write operation performed during a session will be visible in subsequent reads from the same session, without needing to wait for replication or durability checks. In this answer, I'll explain how LSM databases handle read-your-own-writes by focusing on the role of the memtable and how it differs between leader (primary) and follower/replica reads.

An LSM database typically consists of a memtable and multiple immutable sstables. The memtable holds recently written data that is not yet flushed to disk, while sstables are older versions of the data written to persistent storage. When reading within a single session, the read-your-own-writes guarantee can be trivially satisfied by checking the memtable first because it contains all writes made during the current session.

#### Triaging the Question

The key aspects we need to address here include:

Memtable and SSTables: Understanding the role of each in data storage.
Read-Your-Own-Writes Guarantee: Ensuring that writes are visible within a single session.
Leader vs Follower Reads: How read operations differ when performed by the leader versus followers or replicas.
Step-by-Step Solution
1. Memtable and SSTables

In an LSM database, data is initially written to the memtable. When the memtable reaches a certain size or age, it gets flushed to disk as an sstable. Multiple sstables are then merged periodically to optimize read performance.

2. Trivial Satisfaction by Checking Memtable First

When performing reads within a single session in the leader (primary) node:

The memtable is checked first because all writes made during the current session will be present here.
If the data isn’t found in the memtable, subsequent reads check the sstables.

This approach ensures that any write performed during the session is visible in the read operation. This is straightforward and efficient because checking a single in-memory structure (memtable) is faster than searching multiple on-disk structures (sstables).

3. Follower/Replica Reads

For follower or replica nodes, the situation changes slightly:

Since followers do not have direct writes from sessions, they rely entirely on sstables.
When a read operation occurs on a follower, it must check all available sstables to ensure consistency.

In this case, the read-your-own-writes guarantee is more challenging because there's no memtable to provide immediate visibility. Follower nodes need to perform a full scan of their sstable collection to ensure that any writes made during the session are included.

#### Closing Summary
and Common Mistakes
Memtable First Check: Always check the memtable first in leader reads; it simplifies read operations and ensures visibility.
Follower Reads Complexity: On followers, the lack of a memtable forces the system to perform more complex scans across sstables, which can impact performance.

#### Common Mistakes
Ignoring Memtable in Leader Reads: Missing out on the optimization provided by checking the memtable first.
Incorrect Handling of Follower Reads: Assuming that follower reads are identical to leader reads without considering the absence of memtables.
Overlooking Performance Impact: Not recognizing how the absence of a memtable affects read performance in replica nodes.

**Comparison:**

| Memtable vs SSTable Reads |
|--------------------------|----------------------------|
| **Read from Leader**     | **Read from Follower**      |
| - Check memtable first   | - Scan all sstables         |
| - Fast, always finds key | - Slower, may not find key |

**Code:**

```python
# Example code snippet for checking memtable and sstables
def read_data(session_key):
    # Check memtable first (if present)
    if session_key in memtable:
        return memtable[session_key]

    # If not found, check all sstables
    for sstable in sstables:
        value = sstable.get(session_key)
        if value is not None:
            return value

    # Return None if key is not found anywhere
    return None
```

**Example input / output:**

```text
Input: Read data for session_key 1000
Leader Node:
Output: valueA (found in memtable)
Follower Node:
Output: valueA (not found in sstables, memtable not present)

Input: Read data for session_key 5000
Leader Node:
Output: None (not found in memtable or sstables)
Follower Node:
Output: None (not found in any of the sstables)
```

**Data:**

```json
{
  "memtable": {
    "1000": "valueA",
    "2000": "valueB"
  },
  "sstables": [
    { "key": 3000, "value": "valueC" },
    { "key": 4000, "value": "valueD" }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Start Read
6. Is Key in Memtable?
7. Return Value from Memtable
8. Read SSTables?
9. Return First Found Value fromSSTables
10. Key Not Found

