---
type: "practice"
difficulty: "Deep"
---

# Your service caches a value derived from a join across five tables and you cannot cheaply know which rows changed. How would you design invalidation that trades some over-invalidation for guaranteed correctness, and how do you tune that trade-off?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Cache Invalidation](<../../../../07_Distributed_Systems_and_Design/System_Design/Caching/Cache_Invalidation/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this problem, we need to design a caching mechanism that can handle the complexity of joining five tables without knowing exactly which rows have changed. The goal is to ensure correctness while minimizing over-invalidation, where too many cache entries are invalidated unnecessarily. Here’s how we can approach it:

Firstly, let's summarize our key requirements:

Guaranteed Correctness: Ensure that the cached data remains accurate.
Efficiency: Minimize the overhead and latency introduced by caching invalidation mechanisms.

Given these constraints, a common strategy is to use a combination of time-based invalidation and periodic full revalidations. Let's break down the solution step-by-step:

#### Triaging the Question

The core question here is about designing an effective cache invalidation mechanism for data derived from joins across multiple tables. We need to balance between over-invalidation (invalidating too much) and under-invalidation (failing to invalidate when necessary).

#### Assumptions
The five tables are large, making it impractical to query them individually.
Changes in the data can occur at any time.
The service is real-time or near-real-time.

#### First Principles Solution
Time-Based Invalidation: We can set a TTL (time-to-live) on cache entries. This ensures that even if we miss detecting row changes, they will eventually expire from the cache.
Batched Table Updates: Periodically perform a full revalidation of the cached data by joining all five tables and comparing with the current state.
Detailed Steps

Initial Cache Populating:

Perform an initial join across the five tables to populate the cache.

Time-Based Invalidation:

Set TTLs on each cache entry based on how frequently you expect changes to occur. For example, if table updates are infrequent (e.g., once per hour), set a TTL of 30 minutes.

Periodic Full Revalidation:

Schedule periodic full revalidations of the cached data by performing the same join operation.
During these revalidations, compare the old cache state with the new state to identify any changes and update the cache accordingly.

Incremental Invalidations (Optional):

If certain parts of the cache entries are more prone to change than others, consider implementing incremental invalidation mechanisms. For example, if a specific field in one table is frequently updated, invalidate only that part of the cached data.
Tuning the Trade-Off

The key challenge is finding the right balance between over-invalidation and under-invalidation:

Over-Invalidation: Minimizes latency but can lead to inefficiencies.
Under-Invalidation: Ensures correctness but can cause increased read latencies.

To tune this trade-off, consider these strategies:

Frequency of Full Revalidations:

Start with a conservative schedule (e.g., every 2 hours) and gradually increase the frequency based on observed data.

TTL Adjustments:

Shorten or lengthen TTLs based on observed patterns in table updates.

Monitoring and Analytics:

Implement logging to track cache misses and invalidation events.
Use this data to refine your invalidation strategy over time.

#### Closing Summary

In conclusion, a robust caching solution for this scenario involves using a combination of time-based invalidation with periodic full revalidations. This approach ensures correctness while minimizing the risk of unnecessary cache over-invalidation. Tuning this mechanism requires careful monitoring and iterative refinement based on real-world usage patterns.

#### Common Mistakes/Follow-Ups

Mistake: Over-relying solely on TTLs without any revalidation.

#### Follow-Up
Always validate data periodically to ensure correctness.

Mistake: Ignoring the cost of full revalidations.

#### Follow-Up
Balance the frequency and overhead of these operations.

**Code:**

```python
# Example Python code for periodic revalidation

import time
from datetime import timedelta
from database import get_full_state, cache_entry

def validate_cache(tables):
    # Perform full join across tables to get current state
    current_state = get_full_state(tables)

    # Compare with cached data and update as necessary
    for key, value in cache_entry.items():
        if value != current_state.get(key):
            cache_entry[key] = current_state[key]

# Schedule periodic validation
while True:
    validate_cache(['table1', 'table2', 'table3', 'table4', 'table5'])
    time.sleep(timedelta(hours=2).total_seconds())
```

**Example input / output:**

```text
Initial Cache Populating:
- Perform join across tables and populate cache entry.
Periodic Full Revalidation Example Input->Output:
- Input: Trigger revalidation every 2 hours.
- Output: Cache entries updated based on new state from the database.
```

**Data:**

```json
{
  "cache_entry": {
    "key1": "value1",
    "key2": "value2"
  }
}
```

