---
id: "TOOLS/Apache_Kafka/Kafka_Core/Partitions_and_Ordering"
level: 4
type: "concept"
topic: "TOOLS/Apache_Kafka/Kafka_Core"
tools: ["Apache_Kafka"]
---

# Partitions and Ordering

**L4 Advanced** · [↑ Kafka Core](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Apache Kafka](<../_index.md>)

## Practice questions

- [Basic] [How does a producer decide which partition a message goes to?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-30517_How_does_a_producer_decide_which_partition_a_message_goes_to.md>)
- [Intermediate] [Explain how Kafka guarantees message ordering and at what scope ordering holds.](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-30519_Explain_how_Kafka_guarantees_message_ordering_and_at_what_sc.md>)
- [Intermediate] [How would you use a message key to make sure all events for the same order end up in order in your warehouse staging table?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-31045_How_would_you_use_a_message_key_to_make_sure_all_events_for_.md>)
- [Deep] [Why is murmur2 hashing used for default key partitioning, and what breaks if a client uses a different hash?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-30627_Why_is_murmur2_hashing_used_for_default_key_partitioning_and.md>)
- [Deep] [Your anomaly scores must arrive in order per host so a state-machine detector works correctly. How does keying by host_id guarantee this, and what breaks it when you add partitions?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-30906_Your_anomaly_scores_must_arrive_in_order_per_host_so_a_state.md>)
- [Deep] [Why is adding partitions to a live topic risky when your warehouse loads depend on per-key ordering, and what would you do instead?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-31072_Why_is_adding_partitions_to_a_live_topic_risky_when_your_war.md>)
- [Deep] [You need to partition a Kafka-fed data-lake table by event date but Kafka delivers events with up to a day of lateness; how do you avoid rewriting old partitions on every batch?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-31107_You_need_to_partition_a_Kafka-fed_data-lake_table_by_event_d.md>)
- [Deep] [How would you choose the message key so a downstream warehouse merge/upsert stays correct even after you add partitions to the topic later?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-31160_How_would_you_choose_the_message_key_so_a_downstream_warehou.md>)

## Related concepts

- [Partition Keys](<../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Partition_Keys.md>) *(Event Driven Architecture)*
- [Topic Partitioning and Ordering](<../../../03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Topic_Partitioning_and_Ordering.md>) *(Big Data Processing)*
- [Ordering and Partition Keys](<../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Ordering_and_Partition_Keys.md>) *(Distributed Systems Core)*

