---
type: "practice"
difficulty: "Deep"
---

# How would you compute a streaming z-score or EWMA baseline per metric key and emit an alert when the deviation exceeds a threshold?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Multi-granularity Windows](<../../../../TOOLS/Apache_Flink/Flink_Core/Multi-granularity_Windows.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To compute a streaming z-score or Exponentially Weighted Moving Average (EWMA) baseline per metric key and emit an alert based on deviations, we need to implement a stream processing pipeline using Apache Flink. This involves several steps including defining the metrics, computing the statistical measures in a streaming fashion, and setting up the alerting mechanism.

