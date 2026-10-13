---
id: "TOOLS/Apache_Kafka/Kafka_Core/Durability_with_acks_and_ISR"
level: 4
type: "concept"
topic: "TOOLS/Apache_Kafka/Kafka_Core"
tools: ["Apache_Kafka"]
---

# Durability with acks and ISR

**L4 Advanced** · [↑ Kafka Core](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Apache Kafka](<../_index.md>)

## Practice questions

- [Intermediate] [What is replication in Kafka and what are leader and follower replicas?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-30521_What_is_replication_in_Kafka_and_what_are_leader_and_followe.md>)
- [Intermediate] [What is the replication factor and how do you decide its value for a topic?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-30537_What_is_the_replication_factor_and_how_do_you_decide_its_val.md>)
- [Deep] [What is min.insync.replicas and how does it interact with acks=all?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-30563_What_is_min_insync_replicas_and_how_does_it_interact_with_ac.md>)
- [Deep] [What is replica.lag.time.max.ms and how does it decide when a follower falls out of the ISR?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-30659_What_is_replica_lag_time_max_ms_and_how_does_it_decide_when_.md>)
- [Deep] [Why is a rolling broker restart safe for producers with acks=all, and what config makes clients ride through it?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-30708_Why_is_a_rolling_broker_restart_safe_for_producers_with_acks.md>)
- [Deep] [How does min.insync.replicas together with acks=all give you a durability guarantee, and what happens if replicas fall behind?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-31078_How_does_min_insync_replicas_together_with_acks_all_give_you.md>)
- [Deep] [How would you set acks and min.insync.replicas differently for a financial-ledger ingestion topic versus a clickstream analytics topic on the same 3-broker cluster?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-31164_How_would_you_set_acks_and_min_insync_replicas_differently_f.md>)
- [Deep] [You must replicate a telemetry topic from an edge region to a central Kafka for warehouse loading; how does MirrorMaker 2 translate offsets so the central loader resumes correctly on failover?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-31184_You_must_replicate_a_telemetry_topic_from_an_edge_region_to_.md>)

## Related concepts

- [Tiered Storage for Logs](<../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Tiered_Storage_for_Logs.md>) *(Event Driven Architecture)*
- [End to End Exactly Once](<../../../03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/End_to_End_Exactly_Once.md>) *(Big Data Processing)*
- [Consuming Untrusted Streams](<../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Provisional_and_Final_Results/Consuming_Untrusted_Streams.md>) *(Streaming Architecture)*

