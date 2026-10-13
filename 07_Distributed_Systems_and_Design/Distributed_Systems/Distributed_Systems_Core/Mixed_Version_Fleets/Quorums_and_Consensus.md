---
id: "07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Quorums_and_Consensus"
level: 5
type: "deep-dive"
topic: "07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core"
tools: []
---

# Quorums and Consensus

**L5 Expert** · [↑ Distributed Systems Core](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Practice questions

- [Intermediate] [Explain why a consensus-backed database typically runs an odd number of replicas. Walk me through the quorum sizes to tolerate 1 and 2 failures, and why a 3-node cluster survives only one node loss.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-24113_Explain_why_a_consensus-backed_database_typically_runs_an_od.md>)
- [Deep] [Explain at a practical level how Raft keeps a distributed database's replicas consistent: leader election, log replication, and the commit rule. What subtlety around committing entries from a previous term can bite a database's durability guarantee?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-24114_Explain_at_a_practical_level_how_Raft_keeps_a_distributed_da.md>)
- [Deep] [Explain the difference between leader-based (single-writer) replication and leaderless replication. What operational headaches (conflict resolution, write availability, hot leaders) does each impose?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23228_Explain_the_difference_between_leader-based_single-writer_re.md>)
- [Deep] [Explain how a write-ahead log gives durability and enables replication and recovery. Walk through what happens on crash-recovery: redo vs undo, checkpoints, and why fsync matters.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23271_Explain_how_a_write-ahead_log_gives_durability_and_enables_r.md>)
- [Deep] [Explain how a network-partition chaos experiment between two AZs would reveal whether your quorum-based store is genuinely CP. What client behavior on the minority side confirms it's rejecting writes?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23450_Explain_how_a_network-partition_chaos_experiment_between_two.md>)
- [Deep] [Explain how a network-partition chaos experiment against your quorum-replicated database would confirm it's genuinely CP. What behavior on the minority side (write rejection, read fencing) proves it, and how do you scope the experiment so a real bug can't cause customer data loss?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-24246_Explain_how_a_network-partition_chaos_experiment_against_you.md>)
- [Expert] [State the FLP impossibility result in plain terms. If consensus is impossible in a fully asynchronous system with even one faulty node, how do Raft/Paxos work in practice?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23274_State_the_FLP_impossibility_result_in_plain_terms_If_consens.md>)
- [Expert] [You're deciding the org's multi-region write strategy for a system where users are geo-pinned but occasionally roam. Compare home-region routing with conflict handling on move, CRDT-based multi-master, and a global consensus group, and explain how you'd handle a user who moves mid-session.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23545_You_re_deciding_the_org_s_multi-region_write_strategy_for_a_.md>)

## Related concepts

- [HA Failover Design](<../../../../01_Databases_and_SQL/Database_Operations/High_Availability/HA_Failover_Design.md>) *(High Availability)*
- [Concurrent Counters and CAS](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Concurrent_Counters_and_CAS.md>) *(Data Structures and Algorithms)*
- [Distributed Coordination Choices](<../../Concurrency_Patterns/Distributed_Coordination_Choices.md>) *(Concurrency Patterns)*

## References

- *Database Internals* — Alex Petrov — Introduction and Overview › Network Partitions and Partial Failures
- *Database Internals* — Alex Petrov — Introduction and Overview › Two Generals' Problem
- *Database Internals* — Alex Petrov — Introduction and Overview › FLP Impossibility
- *Database Internals* — Alex Petrov — Replication and Consistency › Use CAP Carefully
- *Think Distributed Systems* — Dominik Tornow — Distributed transactions › 6.3 Two-Phase Commit protocol
- *Think Distributed Systems* — Dominik Tornow — Replication › 8.4.5 Single-leader, multileader, and leaderless systems
- *Think Distributed Systems* — Dominik Tornow — Consistency › 9.4 Consistency, availability, and partition tolerance
- *Think Distributed Systems* — Dominik Tornow — Distributed consensus › 10 Distributed consensus
- *Designing Data-Intensive Applications* — Martin Kleppmann — Multi-Leader Replication Topologies
- *Designing Data-Intensive Applications* — Martin Kleppmann — Limitations of Quorum Consistency › Sloppy Quorums and Hinted Handoff
- *Designing Data-Intensive Applications* — Martin Kleppmann — Detecting Concurrent Writes › Version vectors
- *Designing Data-Intensive Applications* — Martin Kleppmann — Unreliable Networks
- *Chaos Engineering: System Resiliency in Practice* — Casey Rosenthal, Nora Jones — Chaos Engineering on a Database › A Real-World Example
- *Ultimate AWS Data Engineering* — Rathish Mohan — Ensuring Consistency Consensus Mechanisms and Models › Introduction
- *Ultimate AWS Data Engineering* — Rathish Mohan — Ensuring Consistency Consensus Mechanisms and Models › The CAP Theorem and Consistency Models
- *Ultimate AWS Data Engineering* — Rathish Mohan — Ensuring Consistency Consensus Mechanisms and Models › Consensus Mechanisms in Distributed Systems
- *Ultimate AWS Data Engineering* — Rathish Mohan — Ensuring Consistency Consensus Mechanisms and Models › Comparing Paxos and Raft
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Data Storage, Indexing, and Replication › Single-Leader

