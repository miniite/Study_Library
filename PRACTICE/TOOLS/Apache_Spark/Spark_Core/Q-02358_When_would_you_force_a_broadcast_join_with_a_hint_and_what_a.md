---
type: "practice"
difficulty: "Deep"
---

# When would you force a broadcast join with a hint, and what are the risks of overriding the optimizer this way?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Shuffle Tuning and Broadcast Joins](<../../../../TOOLS/Apache_Spark/Spark_Core/Shuffle_Tuning_and_Broadcast_Joins.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working as a Data Engineer at an intermediate or advanced level, one of the key optimizations in Apache Spark is understanding when to use broadcast joins. A broadcast join can significantly improve performance by caching small tables in memory on every executor node, rather than shuffling large datasets across the network. However, forcing a broadcast join with a hint should be done carefully, as it can override Spark’s automatic optimization and lead to suboptimal plans if not justified.

This question delves into when you would consider using such a hint and the potential risks associated with doing so. Let's break this down from first principles.

