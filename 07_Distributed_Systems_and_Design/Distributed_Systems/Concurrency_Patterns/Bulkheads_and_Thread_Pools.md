---
id: "07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Bulkheads_and_Thread_Pools"
level: 5
type: "concept"
topic: "07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns"
tools: []
---

# Bulkheads and Thread Pools

**L5 Expert** · [↑ Concurrency Patterns](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Practice questions

- [Intermediate] [Explain the semantics of a semaphore vs a mutex vs a latch. Give a backend use case for each (throttling downstream calls, mutual exclusion, one-time startup gate) and a bug from confusing them.](<../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Q-09871_Explain_the_semantics_of_a_semaphore_vs_a_mutex_vs_a_latch_G.md>)
- [Deep] [Implement a bounded producer-consumer queue by hand. Where exactly do you need condition variables (or wait/notify), and what bug appears if you use notify instead of notifyAll, or forget the wait-in-a-loop rule?](<../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Q-09849_Implement_a_bounded_producer-consumer_queue_by_hand_Where_ex.md>)
- [Deep] [How do you make a singleton / expensive-resource initialization safe when many threads request it at first-touch? Compare eager init, double-checked locking, and a memoizing future so only one thread does the work.](<../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Q-09876_How_do_you_make_a_singleton_expensive-resource_initializatio.md>)
- [Deep] [How would you implement a token-bucket limiter that's concurrency-safe in-process without a global lock per request? Discuss atomic refill math and the race between refill and consume.](<../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Q-09904_How_would_you_implement_a_token-bucket_limiter_that_s_concur.md>)
- [Deep] [How do you architect a fleet-wide capacity plan for a system where concurrency is bounded by a shared distributed lock's throughput ceiling? Show how you'd model the ceiling, forecast when you hit it at projected growth, and the re-sharding roadmap before it becomes a customer-facing incident.](<../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Q-09952_How_do_you_architect_a_fleet-wide_capacity_plan_for_a_system.md>)
- [Expert] [You must increment a per-user counter millions of times per second across many threads. Compare a single atomic, striped/sharded counters (LongAdder-style), and thread-local aggregation. What breaks first at scale?](<../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Q-09867_You_must_increment_a_per-user_counter_millions_of_times_per_.md>)
- [Expert] [You need per-key serialization (all operations for user X run in order, but different users run in parallel). Design a keyed-executor / actor-per-key pattern and discuss how you bound memory when keys are unbounded.](<../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Q-09915_You_need_per-key_serialization_all_operations_for_user_X_run.md>)
- [Expert] [You must design admission control for a multi-tenant platform where one tenant's concurrent burst must never starve others (noisy neighbor). Compare per-tenant bulkheads, weighted fair queuing, and concurrency limits. How do you set and dynamically tune the limits without constant manual intervention?](<../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Q-09986_You_must_design_admission_control_for_a_multi-tenant_platfor.md>)

## Related concepts

- [Lock Granularity Design](<../../System_Design/Low_Level_Design/Compensating_Transactions/Lock_Granularity_Design.md>) *(Low Level Design)*
- [Reviewing AI Code for Performance Risks](<../../../09_Software_Engineering/Developer_Productivity/AI_Assisted_Coding/Reviewing_AI_Code_for_Performance_Risks.md>) *(AI Assisted Coding)*
- [Concurrent Counters and CAS](<../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Concurrent_Counters_and_CAS.md>) *(Data Structures and Algorithms)*

## References

- *Data Engineering Design Patterns* — Bartosz Konieczny — Pattern: Stateful Sessionizer › Scaling
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Flow Design Patterns › Scheduling skew
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Flow Design Patterns › Pattern: Concurrent Runner
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Flow Design Patterns › Solution
- *Data Engineering Best Practices* — Luiz Fernando F. Dos Santos — Infrastructure Operations › Understanding resource contention and allocation strategies
- *Hadoop: The Definitive Guide* — Tom White — YARN › Note
- *Hadoop: The Definitive Guide* — Tom White — YARN › Dominant Resource Fairness
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 9: Execution Plans and Query Store › Evaluating Parallelism Operators
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 12: Azure SQL and Synapse Integration › Workload Management and Resource Classes
- *Database Internals* — Alex Petrov — Introduction and Overview › Chapter 8. Introduction and Overview
- *97 Things Every Data Engineer Should Know* — Tobias Macey (ed.) — Threading and Concurrency Understanding the 2020 Amazon Kinesis Outage › Threading Overhead

