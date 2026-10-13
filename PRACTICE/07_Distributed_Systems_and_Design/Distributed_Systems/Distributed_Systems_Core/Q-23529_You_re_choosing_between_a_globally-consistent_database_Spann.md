---
type: "practice"
difficulty: "Expert"
---

# You're choosing between a globally-consistent database (Spanner/CockroachDB) and a per-region datastore with async replication for the org's next flagship product. Make the principal-level call on the latency-tax of global consensus versus the consistency-tax of regional writes, and where the crossover is.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Sync vs Async Replication](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Sync_vs_Async_Replication.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In choosing between a globally-consistent database like Spanner/CockroachDB and a per-region datastore with async replication for our next flagship product, we must carefully weigh the latency-tax of global consensus against the consistency-tax of regional writes. This decision will significantly impact the overall performance, availability, and cost of the application.

First, let's clarify what each system offers:

Globally-consistent databases like Spanner/CockroachDB enforce strong consistency across all regions, ensuring that every read or write operation is visible globally with minimal latency.
Per-region datastores with async replication allow writes to be processed faster by accepting eventual consistency, but may introduce higher latency and potential data discrepancies.

The key trade-offs revolve around the acceptable level of inconsistency versus the need for strong consistency. We'll reason through this from first principles to determine where these two approaches converge in terms of performance and cost.

Latency-Tax vs Consistency-Tax

Latency-Tax: In a globally-consistent database, every transaction requires coordination across regions, which inherently introduces higher latency. For instance, Spanner aims for cross-region transactions with less than 50ms, while CockroachDB can achieve sub-millisecond latency within a single region.

#### Cons
istency-Tax: With per-region datastores and async replication, writes are faster because they don’t require coordination across regions. However, this approach introduces the risk of temporary inconsistency, which could lead to issues in highly critical applications.

Crossover Points

Critical Transactions vs Non-Critical Operations:

For operations that must be globally consistent (e.g., financial transactions), the latency-tax might be acceptable.
For less critical operations where eventual consistency is tolerable (e.g., logs, analytics data), async replication can offer a performance boost.

Region-Specific Events:

If the application heavily relies on local events and fast response times, per-region storage with async replication could be more suitable.
Conversely, if global operations are equally important, a globally-consistent database is necessary despite higher latency.

Cost vs Performance:

Globally-consistent databases often come with higher costs due to the complexity of maintaining strong consistency across regions.
Per-region datastores might offer lower initial and operational costs but could require additional infrastructure for cross-region coordination if needed.

#### Example Scenario
Global Flagship Product

Assume our flagship product is a real-time collaboration tool used by users from different continents. The core requirement is to maintain low latency for user interactions while ensuring eventual consistency for background operations.

User Interaction: This requires minimal latency and strong consistency, making Spanner/CockroachDB the better choice.
Data Syncing: For syncing data between regions, async replication can be used with periodic reconciliation, reducing overall latency during critical moments.

#### Conclusion

Given our scenario, a hybrid approach might be optimal:

Use a globally-consistent database for critical operations that require strong consistency (e.g., user session management).
Leverage per-region datastores and async replication for non-critical but high-volume data (e.g., real-time chat logs).

This balanced strategy maximizes performance while ensuring the application’s core functionality remains robust.

#### Common Mistakes
and Follow-Ups
Overlooking Localized Events: Assuming all operations can be eventually consistent without considering critical local events.
Ignoring Cost Implications: Failing to account for the higher costs associated with globally-consistent databases, which might not be justifiable for some use cases.
Not Considering Data Reconciliation: In a hybrid approach, failing to implement robust mechanisms for reconciling data between regions can lead to inconsistencies.

**Comparison:**

```
Scenario	Global Consistency Database	Per-region Datastore with Async Replication
Latency for Writes	High (Global Coordination)	Low (Local Coordination)
Latency for Reads	Low (Global Coordination)	High (Async Replication Delays)
Cost	Higher	Lower
Consistency Model	Strong	Eventual
```

**Code:**

```python
# Example of initializing a Spanner client (pseudo-code)
from google.cloud import spanner

def initialize_spanner():
    spanner_client = spanner.Client()
    instance = spanner_client.instance('instance-id')
    database = instance.database('database-id')
    return database
```

**Example input / output:**

```text
Input (Spanner):
- Write a message from user 1 to user 2

Output:
- Message written successfully in the global database with minimal latency.

Input (Per-region Replication):
- User A writes a log entry asynchronously.
- User B reads logs, seeing eventual consistency after some delay.

Output:
- Logs are eventually consistent but faster to write.
```

**Data:**

```json
{
  "users": [
    { "id": 1, "name": "Alice" },
    { "id": 2, "name": "Bob" }
  ],
  "messages": [
    { "senderId": 1, "receiverId": 2, "content": "Hello Bob!" }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Write Request
4. Is Critical Operation?
5. Send to Global DB
6. Send to Local Datastore
7. Process with Minimal Latency,Strong Consistency
8. Async Replication, LowerLatency, Eventual Consistency

