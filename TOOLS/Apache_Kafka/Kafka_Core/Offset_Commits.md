---
id: "TOOLS/Apache_Kafka/Kafka_Core/Offset_Commits"
level: 4
type: "concept"
topic: "TOOLS/Apache_Kafka/Kafka_Core"
tools: ["Apache_Kafka"]
---

# Offset Commits

**L4 Advanced** · [↑ Kafka Core](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Apache Kafka](<../_index.md>)

## Practice questions

- [Basic] [What is an offset in Kafka and how does a consumer use it?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-30514_What_is_an_offset_in_Kafka_and_how_does_a_consumer_use_it.md>)
- [Intermediate] [How would you reset a consumer group's offset to reprocess messages from the beginning?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-30553_How_would_you_reset_a_consumer_group_s_offset_to_reprocess_m.md>)
- [Intermediate] [An interviewer probes: your alert topic occasionally shows offset gaps. Is that a bug in your correlation logic, and what legitimately causes non-contiguous offsets?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-30961_An_interviewer_probes_your_alert_topic_occasionally_shows_of.md>)
- [Intermediate] [How do you keep a consumer group's offsets from being deleted during a long planned maintenance window where consumers are stopped for days?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-30492_How_do_you_keep_a_consumer_group_s_offsets_from_being_delete.md>)
- [Deep] [What is the practical difference between commitSync and commitAsync, and when would you combine both in one consumer?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-30615_What_is_the_practical_difference_between_commitSync_and_comm.md>)
- [Deep] [How would you compute consumer lag from log-end-offset minus committed-offset, and why can raw record count mislead you on rate?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-30703_How_would_you_compute_consumer_lag_from_log-end-offset_minus.md>)
- [Deep] [A consumer processes correctly but its committed offset drifts backward after certain restarts, causing reprocessing. What offset-management bug produces backward drift?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-30759_A_consumer_processes_correctly_but_its_committed_offset_drif.md>)
- [Deep] [How would you design the offset-and-metadata layout so a data engineer can reprocess exactly the events between two wall-clock timestamps after a bad transform deploy?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-31125_How_would_you_design_the_offset-and-metadata_layout_so_a_dat.md>)

## Related concepts

- [Kafka Consumers in Python](<../../Python/Python_Language/Kafka_Consumers_in_Python.md>) *(Python Language)*
- [Partition Keys](<../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Partition_Keys.md>) *(Event Driven Architecture)*
- [Tiered Storage for Logs](<../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Tiered_Storage_for_Logs.md>) *(Event Driven Architecture)*

