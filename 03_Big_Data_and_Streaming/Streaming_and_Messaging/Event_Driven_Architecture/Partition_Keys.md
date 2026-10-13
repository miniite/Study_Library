---
id: "03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Partition_Keys"
level: 4
type: "concept"
topic: "03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture"
tools: ["Apache_Kafka"]
---

# Partition Keys

**L4 Advanced** · [↑ Event Driven Architecture](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Apache Kafka](<../../../TOOLS/Apache_Kafka/_index.md>)

## Practice questions

- [Basic] [What is the difference between a topic, a partition, and an offset in Kafka in one clear sentence each?](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Q-32618_What_is_the_difference_between_a_topic_a_partition_and_an_of.md>)
- [Intermediate] [What is the __consumer_offsets topic in Kafka and why is it stored as a topic rather than in ZooKeeper?](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Q-32620_What_is_the_consumer_offsets_topic_in_Kafka_and_why_is_it_st.md>)
- [Intermediate] [How does Kafka's idempotent producer work and what does enable.idempotence guarantee about duplicates?](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Q-32631_How_does_Kafka_s_idempotent_producer_work_and_what_does_enab.md>)
- [Intermediate] [How would you use a compacted topic to keep the latest state per key, like a materialized cache of user profiles?](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Q-32643_How_would_you_use_a_compacted_topic_to_keep_the_latest_state.md>)
- [Deep] [What is a partition key and how does the choice of key affect message distribution?](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Q-32513_What_is_a_partition_key_and_how_does_the_choice_of_key_affec.md>)
- [Deep] [What is a leader and follower for a Kafka partition, and what happens when a leader fails?](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Q-32522_What_is_a_leader_and_follower_for_a_Kafka_partition_and_what.md>)
- [Deep] [Trace what happens end-to-end when a producer sends a message to Kafka until a consumer processes it.](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Q-32574_Trace_what_happens_end-to-end_when_a_producer_sends_a_messag.md>)
- [Deep] [If you must repartition a topic to add partitions, what breaks about ordering and key-to-partition mapping?](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Q-32630_If_you_must_repartition_a_topic_to_add_partitions_what_break.md>)

## Related concepts

- [Topic Partitioning and Ordering](<../../Distributed_Processing/Big_Data_Processing/Topic_Partitioning_and_Ordering.md>) *(Big Data Processing)*
- [Partitions and Ordering](<../../../TOOLS/Apache_Kafka/Kafka_Core/Partitions_and_Ordering.md>) *(Kafka Core)*
- [Partition Count Decisions](<../../../TOOLS/Apache_Kafka/Kafka_Core/Partition_Count_Decisions.md>) *(Kafka Core)*

