---
type: "practice"
difficulty: "Deep"
---

# A database read that fans out to 100 shards has p99.9 dominated by the slowest shard. Explain the slowest-of-N statistics, and design hedged/tied reads at the data layer, quantifying where the extra query load makes hedging counterproductive.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Sharding Vector Indexes](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Joins_in_MPP_Systems/Sharding_Vector_Indexes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In distributed systems, a common challenge is ensuring high availability and low latency while dealing with uneven performance across multiple shards. The "slowest-of-N" problem arises when a single slow shard can dominate the overall response time of a read operation that needs to fan out across many shards. To address this issue at the data layer, we need to design hedged or tied reads that balance between reducing latency and managing additional query load.

This question delves into understanding how distributed systems handle concurrent requests and how to optimize performance by mitigating the impact of slower shards. We will explore the concept of "slowest-of-N" statistics and then design a strategy for hedging queries, quantifying where adding extra query load becomes counterproductive.

#### Triage
Understanding the Question

The question is asking us to explain the "slowest-of-N" problem in distributed systems and propose solutions that balance between reducing overall latency and managing additional query load. We need to understand the implications of a single shard's performance on the entire system, and then design a mechanism to mitigate this impact.

#### First-Principles Solution

The core idea is to implement hedged reads at the data layer, which means running multiple queries in parallel for each shard and combining their results before returning them to the client. This approach aims to ensure that even if one or more shards are slow, the overall response time remains reasonable.

Step-by-Step Explanation

Understanding Slowest-of-N Statistics:

In a distributed system with 100 shards, if we fan out a read request to all of them, the p99.9 latency is dominated by the slowest shard. This means that even if 99% of the shards respond quickly, the response time will be set by the single slowest shard.
The goal is to ensure that no single slow shard can significantly impact the overall performance.

Designing Hedged Reads:

To achieve this, we can implement a strategy where each read request is sent to multiple shards in parallel.
For example, if we fan out the read to 100 shards but only require one successful response, we might choose to query a subset of these shards (e.g., 20 shards) and return as soon as any one of them responds.

Quantifying Hedging Impact:

We need to determine where adding extra query load becomes counterproductive. This involves calculating the trade-off between reducing latency and increasing query overhead.
For instance, if we hedge by querying 20 shards instead of 100, how much does this impact our overall response time? Conversely, what is the additional cost in terms of query load?

Failure Modes:

If too many shards are hedged, it can lead to increased network traffic and resource utilization.
Conversely, if too few shards are hedged, there is a risk that the slowest shard could still dominate the response time.

#### Closing Summary

By implementing hedged reads, we aim to mitigate the impact of slower shards on overall system performance. However, it's crucial to carefully quantify where this strategy becomes counterproductive in terms of additional query load and increased resource utilization.

#### Common Mistakes/Follow-Ups

Mistake 1: Failing to understand that simply hedging more shards does not always improve performance; there is a trade-off.

#### Follow-Up
Discuss the optimal number of shards to hedge based on empirical data and expected shard performance variability.

Mistake 2: Ignoring the network overhead of running additional queries.

#### Follow-Up
Evaluate the cost-benefit ratio of hedging more shards by measuring both response time improvements and additional query load.

**Comparison:**

```
Shard ID	Response Time (ms)
1	50
2	70
...	...
20	90
21	300
22	350
```

**Code:**

```python
# Example Python code for a simple hedged read implementation

import asyncio
from concurrent.futures import ThreadPoolExecutor

async def fetch_data(shard_id):
    # Simulate data fetching from shard
    await asyncio.sleep(1)  # Simulating network delay
    return f"data from shard {shard_id}"

async def hedged_read(num_shards_to_query, num_shards_total=100):
    with ThreadPoolExecutor(max_workers=num_shards_to_query) as executor:
        loop = asyncio.get_event_loop()
        futures = [loop.run_in_executor(executor, fetch_data, i) for i in range(num_shards_to_query)]
        results = await asyncio.gather(*futures)

    return min(results)  # Return the fastest response
```

**Example input / output:**

```text
Input:
- num_shards_to_query = 20

Output:
- Fastest response from one of the queried shards.
```

**Data:**

```json
{
  "shard_ids": [1, 2, 3, ..., 100]
}
```

**Flow:**

1. Yes
2. No
3. Initiate Read Request
4. Query Which Shards?
5. Select Top N (e.g., 20) Shards
6. Is Any Shard Responding?
7. Return Fastest Response
8. Wait for Next Round ofResponses

