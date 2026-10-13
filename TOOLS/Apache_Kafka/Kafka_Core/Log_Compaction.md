---
id: "TOOLS/Apache_Kafka/Kafka_Core/Log_Compaction"
level: 4
type: "concept"
topic: "TOOLS/Apache_Kafka/Kafka_Core"
tools: ["Apache_Kafka"]
---

# Log Compaction

**L4 Advanced** · [↑ Kafka Core](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Apache Kafka](<../_index.md>)

## Practice questions

- [Basic] [What does it mean that Kafka retains messages for a period even after a consumer reads them and why is that useful for ETL?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-30998_What_does_it_mean_that_Kafka_retains_messages_for_a_period_e.md>)
- [Intermediate] [What is the difference between log retention by time and log compaction?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-30533_What_is_the_difference_between_log_retention_by_time_and_log.md>)
- [Intermediate] [Explain the difference between deleting old data by time retention and keeping only the latest value by compaction, with a data-warehouse example.](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-31033_Explain_the_difference_between_deleting_old_data_by_time_ret.md>)
- [Deep] [How do you handle a Kafka topic that needs to preserve only the latest value per key?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-30606_How_do_you_handle_a_Kafka_topic_that_needs_to_preserve_only_.md>)
- [Deep] [Your alert-noise-reduction service dedupes duplicate alerts using a compacted topic keyed by alert fingerprint. How does compaction help, and what tombstone strategy clears resolved alerts?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-30895_Your_alert-noise-reduction_service_dedupes_duplicate_alerts_.md>)
- [Deep] [Your telemetry topic uses time-based retention but you compact a separate service-state topic. How do you decide which pattern fits which stage of the AIOps pipeline?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-30947_Your_telemetry_topic_uses_time-based_retention_but_you_compa.md>)
- [Deep] [Your compacted topic keeps growing despite compaction being enabled. What operational reasons prevent compaction from reclaiming space and how do you fix each?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-30751_Your_compacted_topic_keeps_growing_despite_compaction_being_.md>)
- [Deep] [You need a compacted topic to serve as an always-current snapshot of a source table that new lake consumers bootstrap from; how do you tune compaction so bootstrap is fast and complete?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-31135_You_need_a_compacted_topic_to_serve_as_an_always-current_sna.md>)

## Related concepts

- [Kafka Retention and Storage](<../../../03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Kafka_Retention_and_Storage.md>) *(Big Data Processing)*
- [Partition Keys](<../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Partition_Keys.md>) *(Event Driven Architecture)*
- [Topic Partitioning and Ordering](<../../../03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Topic_Partitioning_and_Ordering.md>) *(Big Data Processing)*

