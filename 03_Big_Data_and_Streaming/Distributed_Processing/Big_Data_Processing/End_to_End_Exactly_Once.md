---
id: "03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/End_to_End_Exactly_Once"
level: 4
type: "concept"
topic: "03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing"
tools: ["Apache_Kafka", "Apache_Spark"]
---

# End to End Exactly Once

**L4 Advanced** · [↑ Big Data Processing](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Apache Kafka](<../../../TOOLS/Apache_Kafka/_index.md>)
- [Apache Spark](<../../../TOOLS/Apache_Spark/_index.md>)

## Practice questions

- [Basic] [How does Kafka provide durability so that telemetry events are not lost if a consumer is temporarily down?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Q-04599_How_does_Kafka_provide_durability_so_that_telemetry_events_a.md>)
- [Intermediate] [How would you validate that your pipeline is not silently dropping telemetry events somewhere?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Q-04671_How_would_you_validate_that_your_pipeline_is_not_silently_dr.md>)
- [Intermediate] [How would you validate that a pipeline is not silently dropping rows somewhere between source and warehouse?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Q-04876_How_would_you_validate_that_a_pipeline_is_not_silently_dropp.md>)
- [Deep] [Explain how you'd validate that your pipeline isn't silently dropping events, and what reconciliation metrics you'd emit at each stage.](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Q-04725_Explain_how_you_d_validate_that_your_pipeline_isn_t_silently.md>)
- [Deep] [How would you validate an upgrade of Kafka or Spark in a way that gives you confidence the alerting pipeline's semantics didn't subtly change?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Q-04783_How_would_you_validate_an_upgrade_of_Kafka_or_Spark_in_a_way.md>)
- [Deep] [How would you embed data-quality checks (null rates, referential integrity, freshness) into a Spark pipeline so bad batches quarantine instead of publish?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Q-04952_How_would_you_embed_data-quality_checks_null_rates_referenti.md>)
- [Expert] [Defend your choice of min.insync.replicas and acks settings for a critical alerting topic where losing events is worse than a few extra ms of latency.](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Q-04715_Defend_your_choice_of_min_insync_replicas_and_acks_settings_.md>)
- [Expert] [How would you design and test chaos experiments (kill a broker, drop a partition leader, kill an executor) to prove the telemetry pipeline's resilience claims?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Q-04791_How_would_you_design_and_test_chaos_experiments_kill_a_broke.md>)

## Related concepts

- [Kafka Streams and Event Time](<../../../TOOLS/Apache_Kafka/Kafka_Core/Kafka_Streams_and_Event_Time/_index.md>) *(Kafka Core)*
- [Idempotent and Transactional Producers](<../../../TOOLS/Apache_Kafka/Kafka_Core/Idempotent_and_Transactional_Producers.md>) *(Kafka Core)*
- [Consuming Untrusted Streams](<../../Streaming_and_Messaging/Streaming_Architecture/Provisional_and_Final_Results/Consuming_Untrusted_Streams.md>) *(Streaming Architecture)*

