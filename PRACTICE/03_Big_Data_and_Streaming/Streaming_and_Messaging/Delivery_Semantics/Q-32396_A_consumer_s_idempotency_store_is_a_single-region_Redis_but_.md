---
type: "practice"
difficulty: "Expert"
---

# A consumer's idempotency store is a single-region Redis, but the platform is going multi-region active-active. Redesign dedup so it survives region failover without either duplicating or dropping messages during split-brain.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Idempotency and Schema Evolution](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Delivery_Semantics/Idempotency_and_Schema_Evolution.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To redesign the idempotency store to survive region failovers in a multi-region active-active setup, we need to ensure that no message is duplicated or dropped. The core challenge here is managing deduplication across regions while maintaining consistency and avoiding race conditions. The key idea is to leverage distributed consensus mechanisms and a shared state between regions.

TL;DR: Redesign the idempotency store by implementing a distributed deduplication mechanism using a gossip-based protocol like Raft or Paxos, along with a global unique identifier (GUID) for messages. This ensures that messages are processed exactly once across multiple regions, preventing duplicates and drops during split-brain scenarios.

#### Triage

The question is essentially about designing a robust deduplication system in a multi-region active-active setup where the current single-region Redis store can no longer be relied upon due to potential region failovers. The interviewer is probing your understanding of distributed systems, consensus mechanisms, and how to handle split-brain scenarios.

#### Assumptions
Split-Brain Scenario: In an active-active architecture with multiple regions, if a network partition occurs, the regions may enter a split-brain state where they think they are the single active region.

#### Cons
istency Requirements: Messages must be processed exactly once to avoid duplicates and ensure data integrity.

#### First-Principles Solution

GUID for Message Identification:

Assign each message a globally unique identifier (GUID) that can be used to track its delivery status across all regions. This GUID should be generated at the producer side and embedded in every message.

Gossip-Based Consensus Protocol:

Implement a distributed consensus protocol like Raft or Paxos to maintain a single source of truth for the deduplication state among all active regions.
Each region will periodically exchange information about the messages it has seen using this gossip-based protocol, ensuring that no duplicate entries are created.

State Storage:

Store the deduplication states in a globally consistent key-value store such as a distributed database (e.g., Apache Cassandra) to ensure that all regions have access to the latest state.
Use transactions or distributed locks to prevent race conditions when updating the state.

Message Processing Flow:

When a message is received by any region, it checks the global deduplication store for the presence of the GUID.
If the GUID exists and indicates that the message has already been processed in another region, it skips processing; otherwise, it updates the state to mark the message as seen.

Handling Split-Brain:

Implement a mechanism to detect split-brain scenarios using heartbeat mechanisms or other distributed monitoring tools.
Once detected, regions should pause their operations and resume only after converging on a single active state through the consensus protocol.

#### Closing Summary

By implementing a distributed deduplication system using a gossip-based consensus protocol like Raft and storing states in a globally consistent database, we can ensure that messages are processed exactly once across multiple regions. This approach avoids duplicating or dropping messages during split-brain scenarios by maintaining a single source of truth for the deduplication state.

#### Common Mistakes/Follow-Ups
Lack of Consistency: Failing to use a globally consistent data store can lead to inconsistent states and race conditions.
Ignoring Split-Brain Detection: Without proper detection and handling, regions may continue processing messages independently leading to duplicates or drops.
Improper GUID Generation/Management: If the GUIDs are not managed correctly, it could introduce issues such as collisions or incorrect deduplication.

**Comparison:**

```
Region	Deduplication State
A	{"GUID12345": true}
B	{"GUID67890": true}
```

**Code:**

```python
# Example Raft Consensus Protocol Implementation (high-level)
import time

class RaftNode:
    def __init__(self, id):
        self.id = id
        self.state = "follower"
        self.log = []  # Log of commands
        self.current_term = 0
        self.voted_for = None

    def send_heartbeat(self, peers):
        for peer in peers:
            if peer != self.id and self.state == "candidate":
                # Send heartbeat to peer
                pass
```

**Example input / output:**

```text
Input:
- Message with GUID: GUID12345

Output:
RegionA (active):
- Checks state -> GUID12345 exists, skips processing.
RegionB (passive):
- Receives message, checks state -> GUID12345 does not exist, processes and updates state.

Input:
- Message with GUID: GUID67890

Output:
Both regions process the message independently as it is a new message.
```

**Data:**

```json
{
  "regionA": {
    "id": "regionA",
    "deduplicationState": {"GUID12345": true}
  },
  "regionB": {
    "id": "regionB",
    "deduplicationState": {"GUID67890": true}
  }
}
```

**Flow:**

1. Checks GUID
2. Processes if GUID does notexist
3. Processes if GUID exists
4. Region A
5. Region B
6. Mark as seen in state store
7. Skip processing

