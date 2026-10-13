---
id: "07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Cross-service_Invalidation_Races"
level: 5
type: "concept"
topic: "07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns"
tools: ["Redis"]
---

# Cross-service Invalidation Races

**L5 Expert** · [↑ Concurrency Patterns](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Redis](<../../../TOOLS/Redis/_index.md>)

## Practice questions

- [Deep] [You have a read-heavy in-memory cache with rare writes. Compare a plain mutex, a read-write lock, and a copy-on-write approach for this access pattern. Which do you pick and why?](<../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Q-09841_You_have_a_read-heavy_in-memory_cache_with_rare_writes_Compa.md>)
- [Deep] [Explain the readers-writer lock starvation problem (writers or readers starving). How do fair vs unfair RW locks trade throughput for fairness, and which do you default to in a backend service?](<../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Q-09894_Explain_the_readers-writer_lock_starvation_problem_writers_o.md>)
- [Deep] [You need to coordinate a rolling config reload where readers must never see a half-updated multi-field config. Contrast copy-on-write of the whole object vs field-by-field locking, and justify the memory/throughput trade-off.](<../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Q-09934_You_need_to_coordinate_a_rolling_config_reload_where_readers.md>)
- [Expert] [Why isn't a Redis SETNX-based lock safe enough for correctness-critical work, and what does the Redlock debate teach about the difference between a lock for efficiency vs a lock for correctness?](<../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Q-09862_Why_isn_t_a_Redis_SETNX-based_lock_safe_enough_for_correctne.md>)
- [Expert] [A distributed counter (page views) is incremented from many nodes and you need approximate-but-fast reads. Compare CRDT counters, sharded Redis counters, and async aggregation. What consistency do you give up?](<../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Q-09927_A_distributed_counter_page_views_is_incremented_from_many_no.md>)
- [Expert] [Design the concurrency model for a global inventory system where oversell has real dollar cost but strong global serialization would kill throughput at flash-sale scale. How do you partition, reserve, and reconcile so correctness holds while each region runs largely independently?](<../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Q-09940_Design_the_concurrency_model_for_a_global_inventory_system_w.md>)
- [Expert] [A "fix" that added a coarse global lock to stop a rare race silently halved the service's throughput and no one connected the two for weeks. As principal, how do you build the review and observability guardrails so a correctness fix can't quietly become a scalability regression?](<../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Q-09985_A_fix_that_added_a_coarse_global_lock_to_stop_a_rare_race_si.md>)
- [Expert] [A cross-region active-active deployment produces occasional lost updates on the same logical record edited in two regions. Walk through whether you fix this with last-writer-wins, CRDTs, or region-affinity routing, and how you'd explain the "some concurrent edits will conflict" reality to product and support.](<../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Q-09998_A_cross-region_active-active_deployment_produces_occasional_.md>)

## Related concepts

- [Distributed Counters and Consistency](<../Distributed_Systems_Core/Distributed_Joins_in_MPP_Systems/Distributed_Counters_and_Consistency.md>) *(Distributed Systems Core)*
- [Two-level Cache Design](<../../System_Design/Low_Level_Design/Compensating_Transactions/Two-level_Cache_Design.md>) *(Low Level Design)*
- [Optimistic Concurrency and Clocks](<../Distributed_Systems_Core/Distributed_Joins_in_MPP_Systems/Optimistic_Concurrency_and_Clocks.md>) *(Distributed Systems Core)*

## References

- *Database Internals* — Alex Petrov — Transaction Processing and Recovery › CLOCK
- *Database Internals* — Alex Petrov — Distributed Transactions › Distributed Transactions with Calvin
- *Database Internals* — Alex Petrov — T › V
- *Database Internals* — Alex Petrov — T › W
- *Designing Data-Intensive Applications* — Martin Kleppmann — Clock Synchronization and Accuracy › Relying on Synchronized Clocks
- *Designing Data-Intensive Applications* — Martin Kleppmann — Clock Synchronization and Accuracy › Synchronized clocks for global snapshots
- *Designing Data-Intensive Applications* — Martin Kleppmann — B › C
- *Designing Data-Intensive Applications* — Martin Kleppmann — D
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Ordering › Consequences
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Flow Design Patterns › Consequences
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Flow Design Patterns › Shared state
- *Hadoop: The Definitive Guide* — Tom White — ZooKeeper › APIs
- *Hadoop: The Definitive Guide* — Tom White — ZooKeeper › More Distributed Data Structures and Protocols

