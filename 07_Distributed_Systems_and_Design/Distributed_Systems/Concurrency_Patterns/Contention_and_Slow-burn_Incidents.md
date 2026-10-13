---
id: "07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Contention_and_Slow-burn_Incidents"
level: 5
type: "concept"
topic: "07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns"
tools: []
---

# Contention and Slow-burn Incidents

**L5 Expert** · [↑ Concurrency Patterns](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Practice questions

- [Intermediate] [You're reviewing a PR that adds a "quick" synchronized block inside a loop that also does a network call. What's the concurrency red flag, and how would you coach the author to fix it?](<../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Q-09884_You_re_reviewing_a_PR_that_adds_a_quick_synchronized_block_i.md>)
- [Deep] [What is livelock, how does it differ from deadlock, and when have backoff/retry designs caused it? How would you tell them apart from a thread dump alone?](<../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Q-09844_What_is_livelock_how_does_it_differ_from_deadlock_and_when_h.md>)
- [Deep] [Explain the difference between blocking, non-blocking, and asynchronous I/O for a backend that fans out to 50 downstreams per request. How does the concurrency model change tail latency under partial slowness?](<../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Q-09883_Explain_the_difference_between_blocking_non-blocking_and_asy.md>)
- [Deep] [Explain how "async but accidentally sequential" happens — e.g., awaiting inside a loop instead of gathering. How do you spot it in code review and in a trace, and quantify the latency cost?](<../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Q-09900_Explain_how_async_but_accidentally_sequential_happens_e_g_aw.md>)
- [Deep] [You must implement "wait until N events happen, then proceed once." Compare a CountDownLatch, a Phaser/CyclicBarrier, and a counting semaphore. When does the reusable barrier matter and what's the reset race?](<../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Q-09926_You_must_implement_wait_until_N_events_happen_then_proceed_o.md>)
- [Deep] [You're evaluating whether to standardize on optimistic concurrency control across the org's data-access layer. At what write-contention profiles does OCC's retry cost become an architectural liability, and how do you give teams a data-driven rule for OCC vs pessimistic vs partitioned-single-writer?](<../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Q-09974_You_re_evaluating_whether_to_standardize_on_optimistic_concu.md>)
- [Expert] [You migrated a hot code path from synchronous locking to futures/async composition and latency got worse under load. What are the likely causes and how do you profile async code where the stack trace no longer tells the story?](<../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Q-09852_You_migrated_a_hot_code_path_from_synchronous_locking_to_fut.md>)
- [Expert] [Your async pipeline has unbounded queues between stages and memory blows up under a traffic burst. Explain backpressure, and design bounded handoff between producers and consumers that pushes back instead of OOMing.](<../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Q-09875_Your_async_pipeline_has_unbounded_queues_between_stages_and_.md>)

## Related concepts

- [Lock Granularity Design](<../../System_Design/Low_Level_Design/Compensating_Transactions/Lock_Granularity_Design.md>) *(Low Level Design)*
- [Reviewing AI Code for Performance Risks](<../../../09_Software_Engineering/Developer_Productivity/AI_Assisted_Coding/Reviewing_AI_Code_for_Performance_Risks.md>) *(AI Assisted Coding)*
- [Two-level Cache Design](<../../System_Design/Low_Level_Design/Compensating_Transactions/Two-level_Cache_Design.md>) *(Low Level Design)*

## References

- *Data Engineering Design Patterns* — Bartosz Konieczny — Error Management Design Patterns › Consequences
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Flow Design Patterns › Consequences
- *Designing Data-Intensive Applications* — Martin Kleppmann — Process Pauses
- *Designing Data-Intensive Applications* — Martin Kleppmann — Process Pauses › Response time guarantees
- *Designing Data-Intensive Applications* — Martin Kleppmann — Process Pauses › Limiting the impact of garbage collection
- *Hadoop: The Definitive Guide* — Tom White — Crunch › Asynchronous execution
- *Hadoop: The Definitive Guide* — Tom White — ZooKeeper › Recoverable exceptions
- *97 Things Every Data Engineer Should Know* — Tobias Macey (ed.) — Threading and Concurrency Understanding the 2020 Amazon Kinesis Outage › Chapter 82. Threading and Concurrency: Understanding the 2020 Amazon Kinesis Outage

