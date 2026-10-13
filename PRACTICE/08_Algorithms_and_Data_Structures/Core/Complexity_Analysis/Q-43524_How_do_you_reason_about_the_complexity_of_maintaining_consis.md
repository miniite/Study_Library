---
type: "practice"
difficulty: "Deep"
---

# How do you reason about the complexity of maintaining consistency between a primary store and a derived cache, where invalidation is O(dependents) per write, and when a TTL beats explicit invalidation?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Sorting Cost in Services](<../../../../08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Sorting_Cost_in_Services.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To reason about the complexity of maintaining consistency between a primary store and a derived cache, we need to understand how updates propagate from the primary store to the cache, and how invalidations are handled when there's a TTL or explicit invalidation. This involves analyzing both time (T) and space (S) complexities.

The question is essentially asking us to evaluate:

How does the complexity of maintaining consistency change with respect to writes?
What happens when a Time-to-Live (TTL) triggers an automatic cache expiration vs. an explicit invalidation?
How do these operations impact both time and space resources?

We assume:

The primary store is where the authoritative data resides.
A derived cache is used for faster reads, which may be stale due to changes in the primary store.
Each write operation on the primary store involves invalidating the cache entries that depend on it. This invalidation process has a complexity of O(dependents), meaning it depends linearly on how many cache entries are affected.

Let's break down the solution step by step:

Write Operation Complexity:

A write to the primary store will trigger an O(dependents) invalidation operation in the cache.
Each dependent cache entry must be invalidated, which implies a direct proportionality with the number of dependents on that particular piece of data.

TTL vs Explicit Invalidation:

When a TTL is set for cache entries, it introduces an automatic expiration mechanism that can bypass explicit invalidation operations.
If a cache entry's TTL expires before any explicit invalidation occurs, we need to consider both the time to check and update the TTL and the potential additional space required if the entry needs to be recreated.

Time Complexity Analysis:

For each write operation, the time complexity is O(dependents) because of the cache invalidations.
If there are frequent writes (e.g., high transaction rates), this can lead to a higher overall T complexity due to the linear relationship with dependents and number of writes.

Space Complexity Analysis:

Space complexity primarily depends on how many entries are stored in the cache and their sizes.
With TTLs, space is managed more dynamically; entries will naturally expire, reducing overhead. However, this also means that the space might fluctuate based on the current active data set.

#### Trade-offs

Using TTLs can reduce explicit invalidation calls, but it may lead to occasional stale reads.
The choice between TTL and explicit invalidations depends on the trade-off between reducing write overhead (T) and tolerating occasional staleness in read operations.

In summary, maintaining consistency involves a balance of time and space complexities. Writes require O(dependents) invalidation, which can be optimized by using TTLs to reduce explicit invalidations but may lead to stale data. The overall complexity is influenced by the frequency of writes and the number of dependent cache entries.

