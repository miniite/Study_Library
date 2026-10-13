---
type: "practice"
difficulty: "Intermediate"
---

# Your LLM cost dashboard shows average latency is fine but a specific customer complains of slowness. Explain why aggregate latency hides per-tenant tail behavior in token generation and how you'd instrument monitoring to surface it.

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Aggregation Hiding Tail Problems](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Partition_Lag_and_Hot_Keys/Aggregation_Hiding_Tail_Problems.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with distributed systems, especially those involving large language models (LLMs), understanding performance can be complex due to varying workloads across different tenants. The aggregate latency metric can mask issues affecting specific customers or tenants, leading to a situation where overall performance appears satisfactory but individual customers experience significant delays. To address this, we need to delve into the nuances of tail behavior in token generation and devise monitoring strategies that can surface these issues.

