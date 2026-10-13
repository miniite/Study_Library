---
id: "07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Partition_Lag_and_Hot_Keys"
level: 4
type: "concept"
topic: "07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core"
tools: ["Apache_Kafka", "MySQL", "Cassandra", "Kubernetes"]
---

# Partition Lag and Hot Keys

**L4 Advanced** · [↑ Distributed Systems Core](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Go deeper

- [Load Balancing Stateful Inference](<Load_Balancing_Stateful_Inference.md>)
- [Aggregation Hiding Tail Problems](<Aggregation_Hiding_Tail_Problems.md>)

## Implementations (tools)

- [Apache Kafka](<../../../../TOOLS/Apache_Kafka/_index.md>)
- [MySQL](<../../../../TOOLS/MySQL/_index.md>)
- [Cassandra](<../../../../TOOLS/Cassandra/_index.md>)
- [Kubernetes](<../../../../TOOLS/Kubernetes/_index.md>)

## Practice questions

- [Intermediate] [Your document ingestion for RAG uses a stuck Kafka consumer that silently stops updating the index, so users get answers missing recent docs without any error. What SLI would you put on index freshness and how do you distinguish 'slow' from 'stopped'?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23154_Your_document_ingestion_for_RAG_uses_a_stuck_Kafka_consumer_.md>)
- [Intermediate] [Your etcd cluster backing a Kubernetes control plane is throwing 'apply request took too long' warnings and API latency is climbing. Walk me through how you'd triage whether it's disk fsync latency defragmentation need or raft leader instability.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23410_Your_etcd_cluster_backing_a_Kubernetes_control_plane_is_thro.md>)
- [Intermediate] [Your MongoDB replica set stepped down repeatedly during a network hiccup, flapping the primary. Explain the election timeout heartbeat and priority settings behind this and how you'd stabilize it.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23499_Your_MongoDB_replica_set_stepped_down_repeatedly_during_a_ne.md>)
- [Deep] [Your MySQL read-replica lag jumped from 50ms to 40s at peak. Walk me through your debugging: single-threaded replication apply, long-running transactions on the primary, big batch writes, and how you'd confirm the root cause from the replica's status.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-24105_Your_MySQL_read-replica_lag_jumped_from_50ms_to_40s_at_peak_.md>)
- [Deep] [Your replication lag on a MySQL read replica just spiked from 50ms to 40s during peak traffic. Walk me through your debugging steps and the top three root causes you'd check first.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23226_Your_replication_lag_on_a_MySQL_read_replica_just_spiked_fro.md>)
- [Deep] [Explain how a Kubernetes HPA scaling on CPU can oscillate when the workload is actually latency-bound behind a slow dependency. How would you pick a better scaling signal and add stabilization?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23426_Explain_how_a_Kubernetes_HPA_scaling_on_CPU_can_oscillate_wh.md>)
- [Deep] [Your consumer group reading events to compute features is falling behind (lag growing) during peak. Walk me through how you'd diagnose whether it's slow processing, a rebalance storm, or a hot partition, and how you'd add consumer parallelism safely.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23696_Your_consumer_group_reading_events_to_compute_features_is_fa.md>)
- [Deep] [A consumer group rebalance during deploys keeps reprocessing and stalling your ingestion. Explain the rebalance protocol why a slow consumer triggers it and how static membership and cooperative rebalancing reduce the stop-the-world.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-24279_A_consumer_group_rebalance_during_deploys_keeps_reprocessing.md>)

## Related concepts

- [Tail Latency After Expansion](<../../../System_Design/System_Design_Core/Tail_Latency_After_Expansion.md>) *(System Design Core)*
- [Consumer Lag Diagnosis](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Consumer_Lag_Diagnosis.md>) *(Big Data Processing)*
- [Consumer Lag Monitoring](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Consumer_Lag_Monitoring.md>) *(Kafka Core)*

## References

- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Release Management › Replication stalls
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Data Storage, Indexing, and Replication › Replication lag and latency
- *Data Engineering Design Patterns* — Bartosz Konieczny — Time Detectors › Problem
- *Think Distributed Systems* — Dominik Tornow — Replication › 8.4.2 Replication lag
- *Designing Data-Intensive Applications* — Martin Kleppmann — Chapter 8. The Trouble with Distributed Systems
- *Databricks Certified Associate Developer for Apache Spark* — Saba Shah — Questions

