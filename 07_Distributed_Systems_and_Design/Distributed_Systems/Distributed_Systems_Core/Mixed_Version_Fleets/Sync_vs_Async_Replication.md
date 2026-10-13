---
id: "07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Sync_vs_Async_Replication"
level: 5
type: "deep-dive"
topic: "07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core"
tools: []
---

# Sync vs Async Replication

**L5 Expert** · [↑ Distributed Systems Core](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Practice questions

- [Deep] [You must decide synchronous versus asynchronous replication of a fine-tuning checkpoint store across regions for DR. For a job you can't afford to fully restart, argue the durability-versus-latency trade-off and what RPO you'd accept.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23844_You_must_decide_synchronous_versus_asynchronous_replication_.md>)
- [Deep] [Contrast the saga pattern (orchestration vs choreography) with a distributed 2PC transaction. When would you tolerate the eventual-consistency window of a saga, and how do you design compensating actions?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23247_Contrast_the_saga_pattern_orchestration_vs_choreography_with.md>)
- [Deep] [Explain the operational difference between designing for latency-driven geo-replication versus disaster-recovery replication, and how RPO and RTO targets change your topology and the consistency you can offer.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23468_Explain_the_operational_difference_between_designing_for_lat.md>)
- [Expert] [A Patroni-managed Postgres cluster failed over and lost a few committed-looking writes. Explain how synchronous_commit and synchronous_standby_names settings determine whether a promoted replica can lose data, and how you'd tune for zero-RPO.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-24108_A_Patroni-managed_Postgres_cluster_failed_over_and_lost_a_fe.md>)
- [Expert] [You're asked to make a global system survive the loss of an entire cloud region. Compare active-active, active-passive, and pilot-light topologies at the RTO/RPO/cost frontier, and describe how you'd actually rehearse a region evacuation so the plan isn't fiction on the day you need it.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23332_You_re_asked_to_make_a_global_system_survive_the_loss_of_an_.md>)
- [Expert] [You must design the disaster-recovery story for a stateful system with a 15-minute RTO and near-zero RPO across regions. Reconcile that with the physics of cross-region replication latency, and describe how you'd continuously test failover so the RTO is real, plus how you handle failback afterward.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23377_You_must_design_the_disaster-recovery_story_for_a_stateful_s.md>)
- [Expert] [You're choosing between a globally-consistent database (Spanner/CockroachDB) and a per-region datastore with async replication for the org's next flagship product. Make the principal-level call on the latency-tax of global consensus versus the consistency-tax of regional writes, and where the crossover is.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23529_You_re_choosing_between_a_globally-consistent_database_Spann.md>)
- [Expert] [You must set RPO/RTO tiers for the observability estate across metrics, logs, traces, and alerting. Explain why alerting is effectively tier-0 (losing it means flying blind) while cold trace archives can tolerate more loss, and how you'd map replication topology to each tier and stop everyone claiming tier-0.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-24032_You_must_set_RPO_RTO_tiers_for_the_observability_estate_acro.md>)

## Related concepts

- [Replication Topologies and Lag](<../../../../01_Databases_and_SQL/Database_Operations/Replication/Failback_After_Failover/Replication_Topologies_and_Lag.md>) *(Replication)*
- [HA Failover Design](<../../../../01_Databases_and_SQL/Database_Operations/High_Availability/HA_Failover_Design.md>) *(High Availability)*
- [Cross-service Invalidation Races](<../../Concurrency_Patterns/Cross-service_Invalidation_Races.md>) *(Concurrency Patterns)*

## References

- *Ultimate AWS Data Engineering* — Rathish Mohan — Architecting for Scalability Data Replication Techniques › Introduction
- *Ultimate AWS Data Engineering* — Rathish Mohan — Architecting for Scalability Data Replication Techniques › Synchronous versus Asynchronous Replication
- *Ultimate AWS Data Engineering* — Rathish Mohan — Architecting for Scalability Data Replication Techniques › Use Cases and Trade-offs
- *Ultimate AWS Data Engineering* — Rathish Mohan — Architecting for Scalability Data Replication Techniques › Hybrid Replication Models
- *Designing Data-Intensive Applications* — Martin Kleppmann — Chapter 5. Replication › Synchronous Versus Asynchronous Replication
- *Designing Data-Intensive Applications* — Martin Kleppmann — Synchronous Versus Asynchronous Networks
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 10: Transactions, Concurrency, and Isolation Levels › Best Practices
- *Database Internals* — Alex Petrov — D

