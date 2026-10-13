---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd design a distributed rate limiter with a hard org-wide quota that must tolerate a region partition. Why do perfect global limiting and partition tolerance conflict and where do you land pragmatically?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Distributed Rate Limiting](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Rate_Limiting.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

Designing a distributed rate limiter while maintaining global quotas in the presence of regional partitions involves balancing consistency, availability, and partition tolerance (CAP theorem) principles. The solution requires a multi-layered approach that leverages distributed systems concepts such as consistent hashing, token bucket algorithms, and eventual consistency. We must ensure that despite potential partitions, the system can still enforce rate limits and global quotas.

#### Triage

The question is essentially asking how to design a distributed rate limiter in an environment where regional outages might occur but the organization has strict global quota limitations. The core challenge here is ensuring that even when regions are partitioned, the overall system can still adhere to these quotas without violating them.

#### Assumptions
and Core Solution
Global Quota: The organization has a hard limit on the total number of requests allowed globally.
Regional Partitions: There might be scenarios where one or more regions experience network partitions.
Rate Limiting Requirements: Each user or client must have their request rates limited, even if some regions are unavailable.
Detailed Solution
1. Architecture Overview

To design a distributed rate limiter that can handle global quotas while maintaining regional availability, we need to implement the following components:

Token Bucket Filter (TBF): To locally limit the number of requests per user or client.

#### Cons
istent Hashing: To distribute tokens across multiple regions for load balancing and failover.
Eventual Consistency: To manage global quotas in a way that can tolerate partitions.
2. Token Bucket Filter (TBF)

Each client or user is assigned an individual token bucket filter based on their specific rate limits. The TBF ensures that the number of requests per unit time is controlled locally, preventing abuse and overloading downstream services.

Request
Grant or Reject
Client
Local Token Bucket
Service
3. Consistent Hashing

To distribute tokens across multiple regions in a way that maintains consistent distribution even if some regions go down, we use consistent hashing.

Global Tokens
Region A
Region B
Region C
4. Eventual Consistency for Global Quotas

To ensure the global quota is respected, we use a distributed consensus protocol like Raft or Paxos to maintain an eventually consistent view of the total number of requests across all regions.

Log Replication
Log Replication
Log Replication
Region A
Global State
Region B
Region C

#### Implementation
Steps

Initialization:

Set up a distributed database (e.g., Cassandra, Raft) to store and replicate the global quota state across all regions.
Initialize token buckets for each client/user with their respective rate limits.

Request Handling:

When a request comes in, the system first checks if there are tokens available in the local region's TBF.
If not, it uses consistent hashing to determine which global region is responsible for handling this specific user/client.
The responsible region then checks its TBF and updates the global quota state via the distributed consensus protocol.

Handling Partitions:

In case of a partition, regions will continue to operate based on their local TBF but won't update the global state until network conditions improve.
Once connectivity is restored, the system uses the distributed consensus protocol to merge the local states with the current global quota state.

#### Closing Summary

The design involves using token bucket filters and consistent hashing to ensure local rate limiting while leveraging a distributed consensus protocol for eventual consistency in maintaining global quotas. This approach allows the system to maintain availability during regional outages while ensuring that the overall rate limits are not violated.

#### Common Mistakes/Follow-Ups
Mistake 1: Failing to implement consistent hashing, leading to uneven distribution and potential overloading of certain regions.
Mistake 2: Ignoring eventual consistency in global quota management, which can lead to temporary breaches of the quota limits during partitions.

#### Follow-Up
Question: How would you further optimize this system for performance while maintaining its robustness?

**Comparison:**

```
Token Bucket Filter	Consistent Hashing	Raft Consensus Protocol
Local rate limiting	Load balancing	Global state consistency
```

**Code:**

```python
# Example Token Bucket Filter Implementation (TBF)
class TokenBucket:
    def __init__(self, capacity, fill_rate):
        self.capacity = capacity  # Maximum tokens in the bucket
        self.fill_rate = fill_rate  # Tokens added per second
        self.tokens = capacity  # Initialize with full capacity
        self.last_refill_time = time.time()

    def get_token(self):
        now = time.time()
        if now > self.last_refill_time:
            elapsed_time = min(now - self.last_refill_time, 1)
            tokens_to_add = elapsed_time * self.fill_rate
            self.tokens = min(self.capacity, self.tokens + tokens_to_add)
            self.last_refill_time = now
        if self.tokens >= 1:
            self.tokens -= 1
            return True
        else:
            return False

# Example Consistent Hashing Implementation (Pseudo-code)
class ConsistentHashRing:
    def __init__(self, nodes):
        self.ring = {}
        for node in nodes:
            hash_value = hash(node)
            self.ring[hash_value] = node

    def get_node(self, key):
        hash_key = hash(key) % min(len(self.ring), 1024)
        return self.ring.get(hash_key, "default_region")

# Example Raft Consensus Protocol Pseudo-code
class RaftNode:
    def __init__(self, node_id):
        self.node_id = node_id
        self.log = []  # Transaction log

    def append_entries(self, leader_id):
        # Append entries to the log and replicate state changes
```

**Example input / output:**

```text
# Example Input/Output for Token Bucket Filter
tbf = TokenBucket(capacity=2, fill_rate=1)
print(tbf.get_token())  # True
print(tbf.get_token())  # True
print(tbf.get_token())  # False

# Example Input/Output for Consistent Hashing
ring = ConsistentHashRing(["RegionA", "RegionB"])
print(ring.get_node("user1"))  # RegionA
```

**Data:**

```json
{
  "regions": ["RegionA", "RegionB", "RegionC"],
  "globalQuota": {
    "currentUsage": 500,
    "quotaLimit": 1000
  }
}
```

**Flow:**

1. Client
2. Local TBF
3. Grant or Reject
4. Request Served
5. No Token, Use ConsistentHashing
6. Region Determined by CH
7. Check Local TBF and GlobalQuota State via Raft

