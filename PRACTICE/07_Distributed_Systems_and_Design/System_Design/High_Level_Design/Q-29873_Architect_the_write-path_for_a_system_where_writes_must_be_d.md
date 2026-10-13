---
type: "practice"
difficulty: "Expert"
---

# Architect the write-path for a system where writes must be durable across regions before acknowledgment but latency budgets forbid a synchronous cross-region quorum — how do you resolve the tension?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Cross-region Write Paths](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Cross-region_Write_Paths.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To architect a high-level design (HLD) for a write-path in such a system, we need to balance durability with low latency by employing an asynchronous replication strategy with eventual consistency principles. The core idea is to ensure that writes are acknowledged as soon as possible while guaranteeing eventual data durability across regions.

The key aspects to consider include:

Asynchronous Replication: Instead of waiting for a synchronous quorum, the system should asynchronously replicate writes to multiple replicas in different regions.
Eventual Consistency: Introduce a mechanism that ensures data consistency eventually but doesn't compromise on write latency.
Durable Acknowledgment: Ensure that a write is acknowledged only after it has been successfully written locally and at least one remote region.

Let's break this down step-by-step:

Replication Strategy:
The write operation will be sent to the local data center where it is immediately persisted.
Simultaneously, an asynchronous write request is sent to a backup region (e.g., a single replica) for durability purposes.
Local Acknowledgment:
A response is returned to the client as soon as the write is committed locally, even if the remote region hasn't acknowledged it yet.
Remote Replication and Consistency Check:
The backup region will asynchronously replicate the write and periodically check its consistency with the primary region.

This approach ensures that writes are acknowledged quickly while maintaining eventual data durability across regions.

**Comparison:**

```
Write Operation	Acknowledgment Time	Consistency Guarantee
Local DC	Immediate	Not Guaranteed
Remote DC	Asynchronous	Eventually Consistent
```

**Code:**

```python
class WritePath:
    def __init__(self, local_dc, remote_dc):
        self.local_dc = local_dc
        self.remote_dc = remote_dc

    def write(self, data):
        # Step 1: Persist locally
        local_write_id = self.local_dc.persist(data)

        # Step 2: Asynchronously replicate to the backup region
        if self.remote_dc.replicate(data, local_write_id):
            return local_write_id

        # If replication fails, retry or fail fast based on business requirements

    def ensure_consistency(self, write_id):
        # Periodically check consistency in the remote DC
        self.remote_dc.check_consistency(write_id)
```

**Example input / output:**

```text
Write Path Example:
Input: Write "Hello, World!" to the local DC (DC1)
Output: Local DC acknowledges write ID 1

Periodic Check Example:
Input: Ensure consistency for write ID 1 in remote DCs
Output: Remote DC reports successful replication and checks consistency
```

**Data:**

```json
{
  "local_dc": {
    "type": "DataCenter",
    "replicas": ["DC1"]
  },
  "remote_dc": {
    "type": "DataCenter",
    "replicas": ["DC2", "DC3"]
  }
}
```

**Flow:**

1. Success
2. Failure
3. Write Request
4. Local Write Persist
5. Local Acknowledge
6. Remote Replicate
7. Replicated to Remote DCs
8. Retry or Fail Fast
9. Periodic Consistency Check inRemote DCs

