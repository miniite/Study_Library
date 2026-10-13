---
type: "practice"
difficulty: "Deep"
---

# Explain how you would run cache and origin in a canary comparison, reading from both and diffing the results, to catch invalidation bugs before they reach all traffic.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Cache Failover and Resilience](<../../../../07_Distributed_Systems_and_Design/System_Design/Caching/Cache_Invalidation/Cache_Failover_and_Resilience.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To effectively run cache and origin in a canary comparison to catch invalidation bugs before they affect all traffic, we need to set up an A/B testing environment that reads from both the cache and the origin server and compares their responses. The key steps involve understanding the caching mechanism, setting up the canary test, and establishing a robust diffing process.

Firstly, let’s break down what this entails: running two separate flows—caching and origin—side by side for a small subset of traffic (the "canary" group). This involves querying both the cache and the origin server with identical requests and comparing their responses. Any discrepancies will indicate that the cache has an invalid or stale item, which could be due to improper invalidation logic.

**Flow:**

1. Yes
2. No
3. User Request
4. Is User in Canary Group?
5. Fetch From Origin
6. Fetch From Cache
7. Compare with Cache Response
8. Diff Responses
9. Report/Log Difference

