---
type: "practice"
difficulty: "Expert"
---

# You must design the org's approach to read-your-writes consistency for a read-replica-heavy fleet without forcing everything through the leader. Compare sticky routing, write-timestamp tokens, and bounded-staleness reads as a platform primitive teams opt into.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Distributed Counters and Consistency](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Joins_in_MPP_Systems/Distributed_Counters_and_Consistency.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we need to design a read-your-writes consistency model for our distributed system with numerous read replicas while avoiding centralization through the leader node. The key aspects are ensuring data freshness and minimizing network latency, which can be achieved by exploring three main strategies: sticky routing, write-timestamp tokens, and bounded-staleness reads.

Firstly, let's understand what we're trying to achieve: consistency in distributed systems often comes down to balancing between strong (all nodes having the latest data) and eventual (nodes eventually converging on consistent states). The goal here is to find a balance that minimizes the number of writes going through the leader while ensuring reads are up-to-date.

Now, let's dive into each solution step by step:

Sticky Routing

Sticky routing involves directing subsequent read requests from a client back to the same node (or replica) where the last write occurred. This approach is straightforward and can reduce network overhead since it minimizes cross-node communication. However, it doesn’t guarantee strong consistency; writes may still arrive out of order or cause race conditions.

Key Considerations:

#### Pros
Reduces network latency for reads.

#### Cons
May not always reflect the latest state due to potential write-ordering issues.
Use Case: Suitable for scenarios where clients can tolerate occasional staleness.
Write-Timestamp Tokens

A write-timestamp token is a mechanism where each write operation generates a unique timestamp or sequence number. Clients store this token and include it with subsequent read requests. If the latest write has an older timestamp, the system returns a stale result, prompting the client to retry until they receive up-to-date data.

Key Considerations:

#### Pros
Ensures strong consistency by always checking against the latest writes.

#### Cons
Introduces additional overhead for managing and validating tokens; can increase latency if not optimized.
Use Case: Ideal for applications requiring strict transactional integrity, such as financial systems or critical healthcare apps.
Bounded-Staleness Reads

Bounded-staleness reads allow clients to specify a maximum acceptable staleness period. The system then ensures that read operations return data no older than the specified threshold. This approach strikes a balance between strong consistency and performance by trading off some freshness for reduced network load.

Key Considerations:

#### Pros
Flexible and tunable; balances performance with strong consistency.

#### Cons
Requires careful tuning of staleness bounds to avoid too much delay or frequent retries.
Use Case: Suitable for applications where occasional staleness is acceptable, but real-time data updates are important, like social media feeds.
Platform Primitive

Given the diverse needs across different teams, a platform primitive should be designed that allows teams to opt into one of these mechanisms. This ensures flexibility while providing robust consistency guarantees.

Here’s a summary of each approach:

Sticky Routing: Simple and reduces network overhead but may not always reflect the latest state.
Write-Timestamp Tokens: Ensures strong consistency with potential latency costs.
Bounded-Staleness Reads: Flexible and tunable, balancing performance with strong consistency.

#### Closing Summary

In conclusion, we need a platform primitive that supports these mechanisms to cater to varying needs. Sticky routing is best for low-latency requirements but might not be suitable for critical systems needing strict consistency. Write-timestamp tokens provide strong guarantees at the cost of additional overhead, making them ideal for mission-critical applications. Bounded-staleness reads offer a balanced approach, allowing teams to customize their trade-offs between performance and consistency.

#### Common Mistakes
Overlooking write-ordering issues in sticky routing.
Not considering token management complexity in write-timestamp tokens.
Failing to properly tune staleness bounds for bounded-staleness reads.

**Comparison:**

```
Consistency Mechanism	Pros	Cons
Sticky Routing	Reduces network latency	May not reflect latest state
Write-Timestamp Tokens	Strong consistency, transactional integrity	Additional overhead, potential latency
Bounded-Staleness Reads	Flexible tuning, balanced performance and consistency	Requires careful tuning of staleness bounds
```

**Code:**

```python
# Example of a simple sticky routing implementation using Redis
import redis

client = redis.Redis(host='localhost', port=6379, db=0)

def write_data(key, value):
    client.set(key, value)

def read_data(key, last_write_timestamp=None):
    latest_value = client.get(key)

    if last_write_timestamp and latest_value:
        latest_ts = int(latest_value.decode().split(':')[1])
        if latest_ts < last_write_timestamp:
            return f"Stale data: {latest_value}"
    return latest_value
```

**Example input / output:**

```text
Input:
write_data("user_profile", "John Doe")

Output:
True

Input:
read_data("user_profile", 1679504800)

Output:
"John Doe"

Input:
read_data("user_profile", 1679504799)

Output:
"Stale data: John Doe"
```

**Data:**

```json
{
  "key": "user_profile",
  "value": {
    "id": 123,
    "name": "John Doe",
    "timestamp": 1679504800 // Unix timestamp of the last write
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Write Data
6. Is token present?
7. Return latest data
8. Generate new timestamptoken
9. Store and return to client
10. Read Request
11. Use sticky routing?
12. Direct to same node if possible
13. Check with leader for latestdata
14. Staleness Check
15. Return up-to-date data
16. Fallback to bounded stalenesscheck
17. Return data based on stalenessbound or fallback to full sync

