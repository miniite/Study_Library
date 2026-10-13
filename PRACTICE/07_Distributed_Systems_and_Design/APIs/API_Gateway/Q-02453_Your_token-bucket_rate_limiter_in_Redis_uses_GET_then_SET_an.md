---
type: "practice"
difficulty: "Deep"
---

# Your token-bucket rate limiter in Redis uses GET then SET and two concurrent requests both pass when only one should; how do you make the limiter atomic, and what Lua/eval or CRDT approach do you use?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Rate Limiting at the Gateway](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Rate_Limiting_at_the_Gateway.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of a token-bucket rate limiter in Redis where two concurrent requests both pass when only one should, we need to ensure that the operation is atomic. This can be achieved by using Lua scripts or CRDTs (Conflict-Free Replicated Data Types). Here, I'll focus on the Lua script approach as it provides an elegant solution for atomic operations.

The core problem is that the GET and SET commands are not atomic in Redis when used sequentially, leading to race conditions. By wrapping these operations in a Lua script, we can ensure they execute atomically.

