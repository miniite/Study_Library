---
id: "TOOLS/Apache_Kafka/Kafka_Core/Leader_Election_and_Unclean_Failover"
level: 4
type: "concept"
topic: "TOOLS/Apache_Kafka/Kafka_Core"
tools: ["Apache_Kafka"]
---

# Leader Election and Unclean Failover

**L4 Advanced** · [↑ Kafka Core](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Apache Kafka](<../_index.md>)

## Practice questions

- [Basic] [What does the bootstrap.servers list actually do, and why is it fine to list only a subset of brokers?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-30700_What_does_the_bootstrap_servers_list_actually_do_and_why_is_.md>)
- [Deep] [How does leader election happen when a partition leader broker fails?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-30561_How_does_leader_election_happen_when_a_partition_leader_brok.md>)
- [Deep] [What is preferred leader election and why can leaders drift onto a few brokers over time?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-30654_What_is_preferred_leader_election_and_why_can_leaders_drift_.md>)
- [Deep] [An interviewer asks: why did enabling unclean leader election on your telemetry cluster during an outage later corrupt your anomaly baselines, and what's the safer trade-off?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-30970_An_interviewer_asks_why_did_enabling_unclean_leader_election.md>)
- [Deep] [A team enabled unclean.leader.election to restore availability during an outage and later found corrupted downstream data. Explain what happened and the safer path.](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-30506_A_team_enabled_unclean_leader_election_to_restore_availabili.md>)
- [Deep] [A rolling broker upgrade caused producers to see transient NotLeaderForPartition errors. Is this expected, and what client settings make it invisible to your service?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-30753_A_rolling_broker_upgrade_caused_producers_to_see_transient_N.md>)
- [Expert] [What is unclean leader election and why is it risky to enable?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-30562_What_is_unclean_leader_election_and_why_is_it_risky_to_enabl.md>)
- [Expert] [A partition's leader keeps flapping between two brokers under load. How would you diagnose whether it's ISR churn, GC pauses, or network, and what's your fix?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-30779_A_partition_s_leader_keeps_flapping_between_two_brokers_unde.md>)

## Related concepts

- [Leader Election](<../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Idempotent_Ingestion/Leader_Election.md>) *(Distributed Systems Core)*
- [Partition Keys](<../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Partition_Keys.md>) *(Event Driven Architecture)*
- [Tiered Storage for Logs](<../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Tiered_Storage_for_Logs.md>) *(Event Driven Architecture)*

