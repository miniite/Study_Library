---
id: "07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Primary-replica_Databases"
level: 5
type: "deep-dive"
topic: "07_Distributed_Systems_and_Design/System_Design/System_Design_Core"
tools: ["Cassandra", "PostgreSQL"]
---

# Primary-replica Databases

**L5 Expert** · [↑ System Design Core](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Cassandra](<../../../../TOOLS/Cassandra/_index.md>)
- [PostgreSQL](<../../../../TOOLS/PostgreSQL/_index.md>)

## Practice questions

- [Intermediate] [What is a primary-replica (leader-follower) database setup and why use it?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Q-42155_What_is_a_primary-replica_leader-follower_database_setup_and.md>)
- [Deep] [For a news feed would you use fan-out on write or fan-out on read?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Q-42237_For_a_news_feed_would_you_use_fan-out_on_write_or_fan-out_on.md>)
- [Deep] [Explain quorum reads and writes in a Dynamo-style store: with N=3, when does R+W>N guarantee you read your write, and what breaks that guarantee during a partition?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Q-42160_Explain_quorum_reads_and_writes_in_a_Dynamo-style_store_with.md>)
- [Deep] [A read replica is 8 seconds behind the primary and users report reading stale data right after writing. How would you offer read-your-own-writes without forcing all reads to the primary?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Q-42400_A_read_replica_is_8_seconds_behind_the_primary_and_users_rep.md>)
- [Deep] [Your Postgres table hits a deadlock storm under concurrent updates to overlapping rows. How do you read the deadlock graph in the logs and reorder your write access to break the cycle?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Q-42493_Your_Postgres_table_hits_a_deadlock_storm_under_concurrent_u.md>)
- [Deep] [Design cross-region data replication and failover for a Postgres-backed service with an RPO of seconds and an RTO of minutes. What replication mode, and how do you avoid promoting a stale replica?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Q-42550_Design_cross-region_data_replication_and_failover_for_a_Post.md>)
- [Expert] [Your primary election in a Raft-backed store keeps flapping under network jitter. How does Raft pick a leader, why do split-votes happen, and how would you tune election timeouts?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Q-42159_Your_primary_election_in_a_Raft-backed_store_keeps_flapping_.md>)
- [Expert] [Your service does fan-out-on-write for a feed, but a celebrity with 40M followers now stalls the write path for everyone. How would you evolve to a hybrid push-pull model, and what's the read-time cost?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Q-42398_Your_service_does_fan-out-on-write_for_a_feed_but_a_celebrit.md>)

## Related concepts

- [Zero Downtime Sharding Migrations](<../../../Distributed_Systems/Distributed_Systems_Core/Distributed_Joins_in_MPP_Systems/Zero_Downtime_Sharding_Migrations.md>) *(Distributed Systems Core)*
- [Read Your Writes with Replicas](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Read_Your_Writes_with_Replicas.md>) *(Cloud Fundamentals)*
- [Distributed Counters and Consistency](<../../../Distributed_Systems/Distributed_Systems_Core/Distributed_Joins_in_MPP_Systems/Distributed_Counters_and_Consistency.md>) *(Distributed Systems Core)*

## References

- *Database Internals* — Alex Petrov — Anti-Entropy and Dissemination

