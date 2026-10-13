---
id: "TOOLS/Apache_Kafka/Kafka_Core/KRaft_Mode"
level: 5
type: "deep-dive"
topic: "TOOLS/Apache_Kafka/Kafka_Core"
tools: ["Apache_Kafka"]
---

# KRaft Mode

**L5 Expert** · [↑ Kafka Core](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Apache Kafka](<../../_index.md>)

## Practice questions

- [Intermediate] [What is ZooKeeper's role in Kafka and how has KRaft changed this?](<../../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-30520_What_is_ZooKeeper_s_role_in_Kafka_and_how_has_KRaft_changed_.md>)
- [Deep] [Your ingestion cluster runs KRaft now; how does removing ZooKeeper change your operational runbook for controller failover and metadata during a broker outage?](<../../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-31163_Your_ingestion_cluster_runs_KRaft_now_how_does_removing_ZooK.md>)
- [Expert] [How would you migrate a cluster from ZooKeeper to KRaft, and what is the metadata log's role in KRaft?](<../../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-30648_How_would_you_migrate_a_cluster_from_ZooKeeper_to_KRaft_and_.md>)
- [Expert] [In KRaft mode, what is the controller quorum and how does Raft replace the old ZooKeeper-based controller election?](<../../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-30649_In_KRaft_mode_what_is_the_controller_quorum_and_how_does_Raf.md>)
- [Expert] [Explain how KRaft changes your operational runbook for controller failover, and what new failure modes the metadata quorum introduces versus ZooKeeper.](<../../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-30504_Explain_how_KRaft_changes_your_operational_runbook_for_contr.md>)
- [Expert] [How would you architect the KRaft migration for a 200-broker ZooKeeper cluster with zero customer-visible downtime? Define the phased plan the rollback story per phase and the correlated risks the metadata quorum introduces.](<../../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-30800_How_would_you_architect_the_KRaft_migration_for_a_200-broker.md>)

## Related concepts

- [Tiered Storage for Logs](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Tiered_Storage_for_Logs.md>) *(Event Driven Architecture)*
- [Consuming Untrusted Streams](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Provisional_and_Final_Results/Consuming_Untrusted_Streams.md>) *(Streaming Architecture)*
- [Partition Keys](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Partition_Keys.md>) *(Event Driven Architecture)*

