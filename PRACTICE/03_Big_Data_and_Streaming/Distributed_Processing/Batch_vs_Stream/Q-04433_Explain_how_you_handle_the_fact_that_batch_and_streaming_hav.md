---
type: "practice"
difficulty: "Deep"
---

# Explain how you handle the fact that batch and streaming have fundamentally different retry/backpressure semantics. How do you present a single reliability contract to consumers despite that?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Unified Batch and Streaming Semantics](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream/Batch_and_Stream_Reconciliation/Unified_Batch_and_Streaming_Semantics.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with data pipelines, it's common to need to unify both batch and stream processing in order to provide a seamless experience to consumers. The fundamental differences between batch and streaming—specifically in terms of retry semantics and backpressure handling—can pose challenges when trying to present a single reliability contract. To address this, I'll first explain the core concepts, then detail a solution that leverages event sourcing and state machines.

Batch processing typically operates on periodic, fixed-size data sets, often with well-defined start and end times. Streaming, on the other hand, deals with continuous, unbounded streams of data, where backpressure is crucial to avoid overwhelming downstream systems. Retries in batch jobs are usually simpler: if a job fails, it can be retried from the last checkpoint or beginning. In contrast, stream processing often requires more sophisticated retry mechanisms due to its stateful nature and potential for partial failures.

To present a single reliability contract, my approach involves using event sourcing and domain-driven design principles, particularly state machines, which allow us to model both batch and streaming processes in a consistent way.

