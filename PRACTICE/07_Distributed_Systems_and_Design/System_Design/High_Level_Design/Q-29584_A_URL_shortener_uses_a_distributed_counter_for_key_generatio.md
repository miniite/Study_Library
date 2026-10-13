---
type: "practice"
difficulty: "Expert"
---

# A URL shortener uses a distributed counter for key generation — how do you shard the counter across regions without a single point of contention, and how do you avoid gaps or reuse?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Distributed Cache Design](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Distributed_Cache_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a high-level architecture for a URL shortener using a distributed counter that avoids single points of contention and gaps in key generation, we need to consider both consistency and availability. This is a classic problem in distributed systems, where ensuring data integrity across multiple regions while maintaining low latency is crucial.

#### Triaging the Question

The core challenge here involves:

#### Cons
istency: Ensuring that all nodes have access to up-to-date counter values.
Availability: Providing high performance and low latency for URL generation requests from any region.
Avoiding Gaps or Reuse: Preventing the same key from being generated in multiple regions.

#### Assumptions
The system will operate across multiple regions with varying network latencies.
We need to support a large number of short URLs efficiently.
Each region should be able to generate unique keys even when the central server is down or under high load.

#### First Principles Solution
Distributed Counter Design:
Region-Aware Counters: Implement separate counters for each region, ensuring that they are independent and do not interfere with each other.
Synchronization Mechanism:
Use a global coordination service to periodically update local counters in the background.
Conflict Resolution:
In case of conflicts (e.g., two regions generating the same key), use a mechanism like a lease-based approach or a distributed lock.
Step-by-Step Solution
1. Region-Aware Counters

Each region will have its own independent counter to generate short URLs locally. This ensures that no single point of contention exists within any individual region.

Region A Counter
Generate URL Key
Region B Counter
Generate URL Key
2. Synchronization Mechanism (Centralized Service)

A centralized service will periodically update the local counters in each region to ensure they are up-to-date with the latest global sequence number. This can be done using a pull-based approach where regions regularly fetch the current state from the central service.

Central Sync Service
Region A Counter
Region B Counter
3. Conflict Resolution Mechanism

To handle conflicts, use an optimistic concurrency control mechanism like a distributed lock or a lease-based approach.

Distributed Lock: When a region wants to generate a URL key, it tries to acquire a distributed lock. If the lock is not available (indicating that another region has generated the same key), it retries after a short backoff period.
Lease-Based Approach: Each counter can have a lease, and if no new requests are made within the lease time, the counter re-acquires the sequence number from the central service.
Region A Counter
Request Lease
Distribute Sequence Number
Generate URL Key

#### Closing Summary
and Common Mistakes/Follow-Ups
Summary

By implementing region-aware counters, a centralized synchronization service, and robust conflict resolution mechanisms, we can ensure that our URL shortener system is both highly available and free from gaps or key reuse.

#### Common Mistakes
Single Point of Contention: Forgetting to implement separate counters for each region.
Gaps in Key Generation: Not considering the possibility of conflicts when generating keys locally without proper synchronization.
Poor Conflict Resolution: Failing to handle conflicts effectively, leading to duplicated or missing keys.

#### Follow-Ups
Failure Scenarios:
How would your design handle failures in the central sync service?
Scalability:
What happens if we need to scale beyond two regions?
Performance Trade-offs:
How can you optimize network requests between regions for better performance?

**Comparison:**

```
Region	Local Counter	Global Sequence Number
US_EAST	1000	123456789
EU_WEST	500	123456789
AP_SOUTH	300	123456789
```

**Code:**

```python
# Pseudo-code for a simple distributed counter mechanism

class DistributedCounter:
    def __init__(self, region_id, central_sync_service):
        self.region_id = region_id
        self.local_counter = 0
        self.central_sync_service = central_sync_service

    def generate_key(self):
        # Try to acquire lease on the counter
        if not self.acquire_lease():
            return None

        # Fetch the latest global sequence number from the central service
        global_seq_num = self.central_sync_service.get_global_sequence_number()

        # Generate a new key based on local counter and global sequence number
        new_key = f"{global_seq_num}_{self.local_counter}"
        self.local_counter += 1

        return new_key

    def acquire_lease(self):
        # Logic to attempt acquiring the lease, retrying if needed
        pass
```

**Example input / output:**

```text
Input (Request from US_EAST region):
Generate new URL key

Output:
New URL key generated based on synchronized global sequence number and local counter.

Dry-run Request to central sync service for sequence number update:

Request:
{
  "region_id": "US_EAST",
  "sequence_number_request": true,
  "lease_time": 60
}

Response from Sync Service:
{
  "global_sequence_number": 123456789,
  "new_local_counter": 1000
}
```

**Data:**

```json
{
  "region_ids": ["US_EAST", "EU_WEST", "AP_SOUTH"],
  "central_sync_service": {
    "url": "https://sync-service.example.com",
    "interval": 60, // seconds
    "timeout": 5 // seconds
  }
}
```

**Flow:**

1. Region A Counter
2. Request Sequence Number
3. Distribute Sequence Number
4. Generate URL Key
5. Return Generated Key toRequester

