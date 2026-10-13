---
id: "03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Shuffle_Partition_Tuning"
level: 4
type: "concept"
topic: "03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing"
tools: ["Apache_Spark", "Apache_Kafka"]
---

# Shuffle Partition Tuning

**L4 Advanced** · [↑ Big Data Processing](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Apache Spark](<../../../TOOLS/Apache_Spark/_index.md>)
- [Apache Kafka](<../../../TOOLS/Apache_Kafka/_index.md>)

## Practice questions

- [Basic] [What does it mean that Spark performs distributed processing and how does that help with huge log datasets?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Q-04609_What_does_it_mean_that_Spark_performs_distributed_processing.md>)
- [Basic] [Why is Spark called a lazy engine and what benefit does lazy evaluation give when building an ETL job?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Q-04801_Why_is_Spark_called_a_lazy_engine_and_what_benefit_does_lazy.md>)
- [Basic] [What is a partition in Spark (not Kafka) and how does the number of partitions affect how much work runs in parallel?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Q-04820_What_is_a_partition_in_Spark_not_Kafka_and_how_does_the_numb.md>)
- [Intermediate] [What is a shuffle in Spark and why can heavy aggregation on logs be expensive?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Q-04652_What_is_a_shuffle_in_Spark_and_why_can_heavy_aggregation_on_.md>)
- [Intermediate] [What is a broadcast join in Spark and when would joining a small lookup table this way be a good idea?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Q-04826_What_is_a_broadcast_join_in_Spark_and_when_would_joining_a_s.md>)
- [Intermediate] [What is partition pruning and how does reading only the needed date partitions speed up a Spark query?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Q-04873_What_is_partition_pruning_and_how_does_reading_only_the_need.md>)
- [Deep] [What is Delta Lake or a similar table format and why does it help when many jobs read and write the same data?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Q-04888_What_is_Delta_Lake_or_a_similar_table_format_and_why_does_it.md>)
- [Deep] [How would you use foreachBatch to write the same Spark micro-batch atomically to both a warehouse and a cache, and what breaks the atomicity across two sinks?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Q-04909_How_would_you_use_foreachBatch_to_write_the_same_Spark_micro.md>)

## Related concepts

- [Shuffle Tuning and Broadcast Joins](<../../../TOOLS/Apache_Spark/Spark_Core/Shuffle_Tuning_and_Broadcast_Joins.md>) *(Spark Core)*
- [Bucketed Joins in Spark](<../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Bucketed_Joins_in_Spark.md>) *(Pipeline Orchestration)*
- [Spark DataFrames and Lazy Evaluation](<../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Spark_DataFrames_and_Lazy_Evaluation.md>) *(Pipeline Orchestration)*

