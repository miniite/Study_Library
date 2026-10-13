---
id: "07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Zero_Downtime_Sharding_Migrations"
level: 5
type: "deep-dive"
topic: "07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core"
tools: ["PostgreSQL", "MySQL", "Apache_Kafka", "AWS"]
---

# Zero Downtime Sharding Migrations

**L5 Expert** · [↑ Distributed Systems Core](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [PostgreSQL](<../../../../TOOLS/PostgreSQL/_index.md>)
- [MySQL](<../../../../TOOLS/MySQL/_index.md>)
- [Apache Kafka](<../../../../TOOLS/Apache_Kafka/_index.md>)
- [AWS](<../../../../TOOLS/AWS/_index.md>)

## Practice questions

- [Intermediate] [Explain how you'd operate a Redis Cluster across three nodes with replicas and what cluster-require-full-coverage does to availability when a hash-slot range loses both its master and replica.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23432_Explain_how_you_d_operate_a_Redis_Cluster_across_three_nodes.md>)
- [Deep] [Explain the dual-write problem: your service writes to Postgres and publishes to Kafka in the same request. Why can these two writes diverge, and how does the transactional outbox pattern (single DB transaction plus a CDC relay) fix it?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-24067_Explain_the_dual-write_problem_your_service_writes_to_Postgr.md>)
- [Deep] [You added a second region to a Postgres/Cockroach deployment for HA but cross-region writes are now slow. Explain the latency tax of cross-region consensus, and how you'd use region-local tables or follower reads to keep the common path fast.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-24118_You_added_a_second_region_to_a_Postgres_Cockroach_deployment.md>)
- [Deep] [Your CDC pipeline from Postgres via Debezium to Kafka fell behind and then the replication slot bloated the WAL and threatened the primary. Explain this coupling and the operational guardrails you'd add.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23459_Your_CDC_pipeline_from_Postgres_via_Debezium_to_Kafka_fell_b.md>)
- [Deep] [You're running online schema-change tooling (gh-ost/pt-osc/native) on a 2TB table in a sharded MySQL fleet during peak. Explain how it avoids long locks, why replication lag can still spike, and the coordination needed so the change applies consistently across all shards.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-24203_You_re_running_online_schema-change_tooling_gh-ost_pt-osc_na.md>)
- [Expert] [You migrate a warehouse from Redshift to BigQuery and must keep both in sync during the transition. Explain the dual-run reconciliation how you compare aggregates at scale and how you decide the acceptable discrepancy before cutover.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-24307_You_migrate_a_warehouse_from_Redshift_to_BigQuery_and_must_k.md>)
- [Expert] [You must migrate a single-region strongly-consistent DB to multi-region while preserving existing consistency contracts on the endpoints that need them. Explain which reads/writes you can relax to regional, which must stay global, and how you migrate without a correctness regression.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23573_You_must_migrate_a_single-region_strongly-consistent_DB_to_m.md>)
- [Expert] [You're deciding whether a new globally distributed feature should use a CP database that rejects writes on the minority side of a partition or an AP store that accepts them and reconciles later. Give the per-endpoint framework you'd hand product teams so they choose correctly without being distributed-systems experts.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-24165_You_re_deciding_whether_a_new_globally_distributed_feature_s.md>)

## Related concepts

- [HA Failover Design](<../../../../01_Databases_and_SQL/Database_Operations/High_Availability/HA_Failover_Design.md>) *(High Availability)*
- [Primary-replica Databases](<../../../System_Design/System_Design_Core/Clickstream_Counting_Pipeline/Primary-replica_Databases.md>) *(System Design Core)*
- [Replication Topologies and Lag](<../../../../01_Databases_and_SQL/Database_Operations/Replication/Failback_After_Failover/Replication_Topologies_and_Lag.md>) *(Replication)*

## References

- *Cassandra: The Definitive Guide* — Jeff Carpenter, Eben Hewitt — that performs writes to both the old database and the new database. This dual write technique
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Data Storage, Indexing, and Replication › Data Replication
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Data Storage, Indexing, and Replication › Locality
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Data Storage, Indexing, and Replication › Eliminate conflicts
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Data Storage, Indexing, and Replication › Write-anywhere replication
- *Designing Data-Intensive Applications* — Martin Kleppmann — Solutions for Replication Lag › Use Cases for Multi-Leader Replication
- *Designing Data-Intensive Applications* — Martin Kleppmann — Handling Write Conflicts
- *Designing Data-Intensive Applications* — Martin Kleppmann — Handling Write Conflicts › Conflict avoidance
- *Designing Data-Intensive Applications* — Martin Kleppmann — Handling Write Conflicts › Custom conflict resolution logic
- *Data Engineering with Medallion Architecture* — Miki Eto — Medallion Architecture in Multi-cloud › Cautions and limits
- *97 Things Every Data Engineer Should Know* — Tobias Macey (ed.) — Friends Don't Let Friends Do Dual-Writes › Chapter 37. Friends Don't Let Friends Do Dual-Writes
- *Ultimate AWS Data Engineering* — Rathish Mohan — Architecting for Scalability Data Replication Techniques › Using Amazon S3 Cross-Region Replication (CRR) for Backup and Compliance

