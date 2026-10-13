---
type: "practice"
difficulty: "Expert"
---

# How do you architect data governance for a system where concurrent writers can produce anomalies (write-skew, phantom) that violate a regulatory invariant? Where does isolation live, how do you audit that no code path bypasses it, and how do you enforce it across teams?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Lock-free Read Paths](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Lock-free_Read_Paths.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we need to architect data governance in a system where concurrent writers can introduce anomalies like write-skew or phantom reads, which violate regulatory invariants. The key challenge is ensuring that isolation is maintained across concurrent operations and teams while providing auditability. We'll start by defining the problem more precisely and then delve into how to enforce and audit isolation using techniques such as distributed locks, consensus algorithms, and code reviews.

Firstly, let's clarify what we are dealing with. Write-skew occurs when different transactions write conflicting data, leading to inconsistent states. Phantom reads happen when a read query is executed concurrently with writes that could result in the same read query producing different results. These anomalies can violate regulatory requirements if certain invariants are not preserved.

To address these issues, isolation must be enforced at the database level and monitored across teams. This involves using advanced concurrency patterns like two-phase locking (2PL), distributed locks, or consensus algorithms such as Paxos or Raft to coordinate writes. Additionally, we need robust auditing mechanisms to ensure that no code path bypasses these isolation guarantees.

To achieve this:

Define the Regulatory Invariants: Clearly define what invariants are critical for regulatory compliance.
Choose Concurrency Patterns: Select appropriate concurrency control patterns like 2PL or distributed locks.
Implement Auditing Mechanisms: Use logging and monitoring to track and verify that isolation is being enforced.
Enforce Consistency Across Teams: Ensure all teams follow the same rules through code reviews and automated tests.

Next, we'll detail each of these steps with a concrete example using an in-memory database like Redis for illustration purposes.

**Comparison:**

```
Concurrency Pattern	Description
Two-phase locking	Ensures that transactions are either fully committed or rolled back
Distributed locks	Manages concurrency across multiple nodes
Consensus algorithms	Achieve agreement among distributed processes
```

**Code:**

```python
# Example of implementing distributed locks using Redis
import redis

def acquire_lock(key):
    r = redis.Redis(host='localhost', port=6379, db=0)
    return r.setnx(key, "locked")

def release_lock(key):
    r = redis.Redis(host='localhost', port=6379, db=0)
    if r.get(key) == b'locked':
        r.delete(key)

# Example of using the lock in a function
def write_data(key, value):
    if acquire_lock(key):
        try:
            # Perform the write operation
            data = load_data(key)
            data.append(value)
            save_data(key, data)
        finally:
            release_lock(key)
```

**Example input / output:**

```text
Input: write_data("inventory", "item-123")
Output:
1. Lock acquired for key "inventory".
2. Data read: ["existing-item-456"]
3. New data written: ["existing-item-456", "item-123"]
4. Lock released.
```

**Data:**

```json
{
  "data": {
    "key": "inventory",
    "value": "item-123"
  }
}
```

**Flow:**

1. Yes
2. No
3. Transaction Starts
4. Acquire Lock
5. Perform Write Operation
6. Commit Transaction
7. Release Lock
8. Rollback Transaction

