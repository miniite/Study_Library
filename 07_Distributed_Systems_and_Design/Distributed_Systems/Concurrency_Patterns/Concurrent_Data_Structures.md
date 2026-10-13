---
id: "07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Concurrent_Data_Structures"
level: 5
type: "concept"
topic: "07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns"
tools: []
---

# Concurrent Data Structures

**L5 Expert** · [↑ Concurrency Patterns](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Practice questions

- [Deep] [Explain the difference between a data race and a race condition. Give a concrete example where code has a race condition but no data race, and one with a data race but arguably correct output.](<../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Q-09838_Explain_the_difference_between_a_data_race_and_a_race_condit.md>)
- [Deep] [What guarantees does an actor model give you about state that the lock-based model doesn't, and where does it still let races in (e.g., across actors, message reordering)? When would you choose actors for a backend service?](<../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Q-09851_What_guarantees_does_an_actor_model_give_you_about_state_tha.md>)
- [Deep] [Explain what "thread confinement" buys you and how single-threaded event loops (Node/Redis/Nginx) sidestep most data races. What new class of concurrency bugs do they introduce instead?](<../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Q-09866_Explain_what_thread_confinement_buys_you_and_how_single-thre.md>)
- [Deep] [When would you pick a concurrent data structure (ConcurrentHashMap, lock-free queue) over external locking around a plain one? What do the concurrent structures NOT protect (compound operations) that still bite people?](<../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Q-09882_When_would_you_pick_a_concurrent_data_structure_ConcurrentHa.md>)
- [Deep] [Design a concurrent in-memory job dedup: while a job for key K is running, new requests for K should attach to the running one, not start a second. Where's the race between "check running" and "register running"?](<../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Q-09908_Design_a_concurrent_in-memory_job_dedup_while_a_job_for_key_.md>)
- [Deep] [You're asked to make "race-condition classes" a measurable quality metric across the org (like a defect taxonomy). How would you categorize concurrency defects, instrument their frequency from incidents, and use that data to prioritize which paved-road guardrail investments cut the most risk?](<../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Q-10033_You_re_asked_to_make_race-condition_classes_a_measurable_qua.md>)
- [Expert] [Distinguish lock-free from wait-free algorithms. Give a scenario where lock-free is enough and one where wait-free is required, and explain the throughput/latency trade-off you accept with each.](<../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Q-09853_Distinguish_lock-free_from_wait-free_algorithms_Give_a_scena.md>)
- [Expert] [Explain memory-ordering modes (relaxed/acquire-release/sequentially-consistent) with a concrete producer flag + data example. Which is the minimum ordering that makes the consumer safely observe the data?](<../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Q-09860_Explain_memory-ordering_modes_relaxed_acquire-release_sequen.md>)

## Related concepts

- [Thread Safety and Locking](<../../../TOOLS/DotNet/DotNet_Platform/Thread_Safety_and_Locking.md>) *(DotNet Platform)*
- [Lock Granularity Design](<../../System_Design/Low_Level_Design/Compensating_Transactions/Lock_Granularity_Design.md>) *(Low Level Design)*
- [Reviewing AI Code for Performance Risks](<../../../09_Software_Engineering/Developer_Productivity/AI_Assisted_Coding/Reviewing_AI_Code_for_Performance_Risks.md>) *(AI Assisted Coding)*

## References

- *Think Distributed Systems* — Dominik Tornow — Thinking in distributed systems Models, mindsets, and mechanics › 1.8 Thinking above the code
- *Think Distributed Systems* — Dominik Tornow — System models, order, and time › 2.1.6 Realistic system models
- *Think Distributed Systems* — Dominik Tornow — Transactions › 5.2.1 Concurrency
- *Designing Data-Intensive Applications* — Martin Kleppmann — Event Sourcing › Concurrency control
- *Cracking the Data Engineering Interview* — Kedeisha Bryan, Taamir Ransome — Technical interview questions › General technical concept questions
- *Data Engineering Design Patterns* — Bartosz Konieczny — C

