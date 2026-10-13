---
type: "practice"
difficulty: "Expert"
---

# Design a double-entry ledger service where every write must be atomic across debit and credit accounts at 20K TPS — how do you shard accounts while still keeping cross-account transfers consistent?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Payment Ledgers](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Payment_Ledgers.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a high-level ledger service that supports 20,000 TPS (transactions per second) and ensures atomicity across debit and credit accounts during cross-account transfers, we need to address both performance and consistency challenges. The core idea is to shard the accounts while maintaining global visibility for transactions. Here’s how we can achieve this:

#### Triaging the Question

The question is essentially asking us to design a distributed ledger system that handles high write throughput (20K TPS) while ensuring atomicity of cross-account transactions. The interviewer will be probing our understanding of distributed systems, sharding strategies, consistency models, and trade-offs between performance and data integrity.

#### Assumptions
Account Structure: Accounts are uniquely identifiable by their IDs.
Transaction Types: Each transaction consists of a debit and credit operation (cross-account transfer).

#### Cons
istency Model: We need to support eventual or strong consistency models depending on the use case, but we must ensure atomicity.
TPS Requirement: 20K TPS is a high throughput requirement that necessitates efficient sharding and scaling strategies.

#### Solution
Step-by-Step First Principles Approach

Sharding Strategy:

Account Sharding: Divide accounts into shards based on account ID hash to ensure even distribution.
Cross-Shard Consistency: Ensure consistency across shard boundaries for cross-account transfers using distributed locks or consensus protocols.

System Architecture:

Leader-Follower Model: Design a leader-follower model where each shard has a leader that handles write operations and replicas handle reads and secondary writes.

#### Cons
ensus Protocol: Use a consensus protocol like Raft to ensure strong consistency among shards during cross-account transactions.
Distributed Locks: Implement distributed locks (e.g., using Redis or etcd) to manage concurrent access during cross-shard transactions.

Transaction Handling:

Local Transactions: Perform local debit and credit operations on the leader of the respective shards.
Global Consistency Check: Before committing the transactions, ensure atomicity by acquiring distributed locks or using a consensus protocol to coordinate across shards.

Performance Optimization:

Caching: Use caching (e.g., Redis) to reduce database load and improve read performance.
Batching: Batch multiple writes to reduce network overhead and improve overall throughput.

Failure Handling:

Replica Synchronization: Ensure replicas are kept in sync with the leader using a raft-based protocol.
Fault Tolerance: Design the system to handle failures gracefully, ensuring that transactions can be retried or coordinated across shards.

#### Closing Summary

In summary, designing a double-entry ledger service for 20K TPS involves a combination of sharding accounts, implementing distributed locks and consensus protocols, using a leader-follower model with Raft for consistency, and optimizing performance through caching and batching. The key is to balance global consistency requirements with high write throughput by carefully managing cross-shard transactions.

#### Common Mistakes/Follow-Ups
Ignoring Distributed Locks: Forgetting the importance of distributed locks in maintaining atomicity during cross-account transfers.
Not Using a Consensus Protocol: Failing to implement a consensus protocol like Raft for ensuring strong consistency across shards.
Overlooking Failure Handling: Ignoring how to handle failures and ensure transactional integrity when shards fail.

By addressing these points, we can build a robust and scalable ledger service that meets the high TPS requirement while maintaining atomicity and consistency.

**Comparison:**

```
Shard	Leader	Replicas
0	Node A	Node B, Node C
1	Node D	Node E, Node F
```

**Code:**

```python
# Example of a simplified Raft-based consensus protocol for shard coordination
import time

class ShardLeader:
    def __init__(self):
        self.log = []  # Transaction log
        self.currentTerm = 0
        self.votedFor = None
        self.commitIndex = 0
        self.lastApplied = 0

    def appendEntry(self, entry):
        self.log.append(entry)
        if len(self.log) > self.commitIndex:
            self.commitIndex += 1
```

**Example input / output:**

```text
Input:
- Transaction from account 1 to account 2 for $100

Output:
- Account 1 balance after transaction: 400
- Account 2 balance after transaction: 800
- Consistency ensured across shards
```

**Data:**

```json
{
  "accounts": [
    {"id": 1, "balance": 500},
    {"id": 2, "balance": 700}
  ],
  "transactions": [
    {"from_id": 1, "to_id": 2, "amount": 100},
    {"from_id": 2, "to_id": 1, "amount": 50}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Transaction Request
4. Is Local Transaction?
5. Local Commit
6. Distributed Lock Acquire
7. Consensus Check
8. Global Consistency Check
9. Commit to Leader and Replicas
10. Notify Client Success

