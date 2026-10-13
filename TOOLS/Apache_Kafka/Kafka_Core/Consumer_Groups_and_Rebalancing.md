---
id: "TOOLS/Apache_Kafka/Kafka_Core/Consumer_Groups_and_Rebalancing"
level: 4
type: "concept"
topic: "TOOLS/Apache_Kafka/Kafka_Core"
tools: ["Apache_Kafka"]
---

# Consumer Groups and Rebalancing

**L4 Advanced** · [↑ Kafka Core](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Apache Kafka](<../_index.md>)

## Practice questions

- [Basic] [What is a consumer group and how does it enable parallel consumption?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-30515_What_is_a_consumer_group_and_how_does_it_enable_parallel_con.md>)
- [Intermediate] [What happens if you have more consumers in a group than partitions?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-30539_What_happens_if_you_have_more_consumers_in_a_group_than_part.md>)
- [Intermediate] [How does a consumer fetch from multiple partitions in one request, and how are records returned across those partitions?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-30666_How_does_a_consumer_fetch_from_multiple_partitions_in_one_re.md>)
- [Intermediate] [Your consumer is single-threaded and can't keep up. Before adding partitions, what simpler wins would you look for first?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-31065_Your_consumer_is_single-threaded_and_can_t_keep_up_Before_ad.md>)
- [Intermediate] [A consumer joins a group and immediately gets assigned zero partitions while others are overloaded. What assignment-strategy or membership issue explains this?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-30749_A_consumer_joins_a_group_and_immediately_gets_assigned_zero_.md>)
- [Deep] [How do you pause and resume specific partitions on a consumer, and give a real scenario where you would need to.](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-30619_How_do_you_pause_and_resume_specific_partitions_on_a_consume.md>)
- [Deep] [Why is one consumer thread per partition a common concurrency model, and what breaks if you share a consumer across threads?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-30667_Why_is_one_consumer_thread_per_partition_a_common_concurrenc.md>)
- [Deep] [How would you detect and safely recover from a stuck consumer that holds a partition assignment but has silently stopped making progress mid-poll?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-30729_How_would_you_detect_and_safely_recover_from_a_stuck_consume.md>)

## Related concepts

- [Topic Partitioning and Ordering](<../../../03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Topic_Partitioning_and_Ordering.md>) *(Big Data Processing)*
- [Partition Keys](<../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Partition_Keys.md>) *(Event Driven Architecture)*
- [Tiered Storage for Logs](<../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Tiered_Storage_for_Logs.md>) *(Event Driven Architecture)*

