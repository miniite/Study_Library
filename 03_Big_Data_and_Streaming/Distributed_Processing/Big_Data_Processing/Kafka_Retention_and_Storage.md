---
id: "03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Kafka_Retention_and_Storage"
level: 4
type: "concept"
topic: "03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing"
tools: ["Apache_Kafka", "Apache_Spark"]
---

# Kafka Retention and Storage

**L4 Advanced** · [↑ Big Data Processing](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Apache Kafka](<../../../TOOLS/Apache_Kafka/_index.md>)
- [Apache Spark](<../../../TOOLS/Apache_Spark/_index.md>)

## Practice questions

- [Basic] [In simple terms what is a Kafka topic and how would you use one to carry application logs?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Q-04606_In_simple_terms_what_is_a_Kafka_topic_and_how_would_you_use_.md>)
- [Basic] [What is retention in Kafka and how would you decide how long to keep a logs topic?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Q-04622_What_is_retention_in_Kafka_and_how_would_you_decide_how_long.md>)
- [Basic] [What is Kafka retention and how would you decide how long to keep events on a topic?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Q-04840_What_is_Kafka_retention_and_how_would_you_decide_how_long_to.md>)
- [Intermediate] [How would you reduce the volume of metrics before they hit long-term storage without losing key signals?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Q-04653_How_would_you_reduce_the_volume_of_metrics_before_they_hit_l.md>)
- [Intermediate] [Why is compression useful for Kafka messages carrying verbose logs and what is the trade-off?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Q-04674_Why_is_compression_useful_for_Kafka_messages_carrying_verbos.md>)
- [Deep] [Your topic retention is 3 days but a bug corrupted a downstream store 5 days ago; walk me through your options and what you'd change so it doesn't recur.](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Q-04712_Your_topic_retention_is_3_days_but_a_bug_corrupted_a_downstr.md>)
- [Deep] [How do you decide compaction versus time-based retention for a topic that carries the latest health state per host?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Q-04722_How_do_you_decide_compaction_versus_time-based_retention_for.md>)
- [Deep] [Your broker disks are filling faster than expected on a logs topic; how do you decide between more retention tuning, compression, tiered storage, or sampling?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Q-04735_Your_broker_disks_are_filling_faster_than_expected_on_a_logs.md>)

## Related concepts

- [Log Compaction](<../../../TOOLS/Apache_Kafka/Kafka_Core/Log_Compaction.md>) *(Kafka Core)*
- [Claim Check Pattern](<../../../TOOLS/Apache_Kafka/Kafka_Core/Claim_Check_Pattern.md>) *(Kafka Core)*
- [Topic Design for Re-sync](<../../../TOOLS/Apache_Kafka/Kafka_Core/Topic_Design_for_Re-sync.md>) *(Kafka Core)*

