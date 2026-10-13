---
id: "07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Locks_and_Fencing"
level: 5
type: "deep-dive"
topic: "07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core"
tools: ["Redis"]
---

# Distributed Locks and Fencing

**L5 Expert** · [↑ Distributed Systems Core](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Redis](<../../../../TOOLS/Redis/_index.md>)

## Practice questions

- [Deep] [You need a distributed lease so exactly one worker runs a scheduled nightly reindex even if the holder GC-pauses. Explain why a plain lock is unsafe here and where the fencing token must be checked to prevent a double reindex.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23165_You_need_a_distributed_lease_so_exactly_one_worker_runs_a_sc.md>)
- [Deep] [Explain why a distributed lock is often the wrong abstraction for a database job and when you'd instead make the operation idempotent or use a fencing token. Give a concrete SQL example for each choice.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-24095_Explain_why_a_distributed_lock_is_often_the_wrong_abstractio.md>)
- [Deep] [Explain the concept of a lease and how leases give you a time-bounded exclusive right that's safer than a lock. Why must the holder do work only within the lease and re-check on renewal?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23295_Explain_the_concept_of_a_lease_and_how_leases_give_you_a_tim.md>)
- [Deep] [Explain how you'd design a distributed lock or lease so exactly one worker in your fleet runs a given scheduled retraining job, tolerating the holder GC-pausing. Why is a lease with a fencing token safer than a plain lock here?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23689_Explain_how_you_d_design_a_distributed_lock_or_lease_so_exac.md>)
- [Deep] [Explain how you'd design a distributed scheduler that runs each team's scheduled retraining exactly once at its scheduled time across a fleet, tolerating node failure, and prevents a thundering herd of retrains all firing at midnight from swamping the shared GPU cluster.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23787_Explain_how_you_d_design_a_distributed_scheduler_that_runs_e.md>)
- [Deep] [Explain how you'd design a distributed lock or lease backed by etcd/ZooKeeper so exactly one worker runs a critical database maintenance job, why you'd prefer a lease with a fencing token over a plain lock when the holder can GC-pause, and where the fence must be checked.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-24232_Explain_how_you_d_design_a_distributed_lock_or_lease_backed_.md>)
- [Expert] [Explain why a naive 'leader = node holding a lock in ZooKeeper' design can still allow two nodes to believe they're leader. What is a fencing token and where exactly must it be checked?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23218_Explain_why_a_naive_leader_node_holding_a_lock_in_ZooKeeper_.md>)
- [Expert] [When is a distributed lock the wrong abstraction, and when should you instead use a lease, a fencing token, or make the operation idempotent? Give a concrete example for each.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23267_When_is_a_distributed_lock_the_wrong_abstraction_and_when_sh.md>)

## Related concepts

- [Lock Granularity Design](<../../../System_Design/Low_Level_Design/Compensating_Transactions/Lock_Granularity_Design.md>) *(Low Level Design)*
- [Cross-service Invalidation Races](<../../Concurrency_Patterns/Cross-service_Invalidation_Races.md>) *(Concurrency Patterns)*
- [Distributed Coordination Choices](<../../Concurrency_Patterns/Distributed_Coordination_Choices.md>) *(Concurrency Patterns)*

## References

- *Designing Data-Intensive Applications* — Martin Kleppmann — Knowledge, Truth, and Lies › Fencing tokens
- *Designing Data-Intensive Applications* — Martin Kleppmann — Knowledge, Truth, and Lies › Byzantine Faults

