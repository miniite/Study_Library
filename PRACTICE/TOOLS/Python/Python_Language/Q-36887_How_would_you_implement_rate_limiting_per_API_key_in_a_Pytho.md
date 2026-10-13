---
type: "practice"
difficulty: "Deep"
---

# How would you implement rate limiting per API key in a Python gateway using a sliding window or token bucket, and where do you store counters for correctness across replicas?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Rate-limited Integrations](<../../../../TOOLS/Python/Python_Language/Rate-limited_Integrations.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To implement rate limiting per API key using a sliding window or token bucket approach in a Python gateway, we need to ensure that each request from an API key is properly throttled based on predefined limits. The solution involves storing counters across replicas to maintain correctness and consistency.

The core idea is to use a data structure like a token bucket or a sliding window to track the number of requests for each API key within a certain time frame. We will store these counters in a distributed storage system, such as Redis, to ensure they are consistent across multiple replicas.

