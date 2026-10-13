---
id: "03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Spark_Executor_Sizing"
level: 4
type: "concept"
topic: "03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing"
tools: ["Apache_Spark", "Apache_Kafka"]
---

# Spark Executor Sizing

**L4 Advanced** · [↑ Big Data Processing](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Apache Spark](<../../../TOOLS/Apache_Spark/_index.md>)
- [Apache Kafka](<../../../TOOLS/Apache_Kafka/_index.md>)

## Practice questions

- [Basic] [In one sentence what is a Spark job and what kind of telemetry work would you run as one?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Q-04602_In_one_sentence_what_is_a_Spark_job_and_what_kind_of_telemet.md>)
- [Intermediate] [Your Spark job is running much slower than you expected; what basic things would you check first?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Q-04837_Your_Spark_job_is_running_much_slower_than_you_expected_what.md>)
- [Deep] [What trade-offs would you weigh between more partitions for parallelism versus the overhead they add?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Q-04686_What_trade-offs_would_you_weigh_between_more_partitions_for_.md>)
- [Deep] [A Spark executor keeps getting killed by the cluster OOM-killer during a windowed percentile computation; how do you find and fix the memory pressure?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Q-04716_A_Spark_executor_keeps_getting_killed_by_the_cluster_OOM-kil.md>)
- [Deep] [An interviewer asks: what's the trade-off between raising alerts inside the Spark job versus emitting scored signals to a separate alert engine?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Q-04760_An_interviewer_asks_what_s_the_trade-off_between_raising_ale.md>)
- [Deep] [How would you detect and fix a broadcast join that silently degraded to a sort-merge join because the small side grew past the broadcast threshold?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Q-04915_How_would_you_detect_and_fix_a_broadcast_join_that_silently_.md>)
- [Deep] [How would you decide whether autoscaling a Spark cluster is worth it for a bursty streaming workload versus running fixed capacity?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Q-04948_How_would_you_decide_whether_autoscaling_a_Spark_cluster_is_.md>)
- [Expert] [Your Structured Streaming job's state store grows to hundreds of GB during a stream-stream join; how do you bound it and what does the watermark actually control here?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Q-04906_Your_Structured_Streaming_job_s_state_store_grows_to_hundred.md>)

## Related concepts

- [Profiling Spark Stages](<../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Profiling_Spark_Stages.md>) *(Pipeline Orchestration)*
- [Spark Execution Caching and Debugging](<../../../TOOLS/Apache_Spark/Spark_Core/Spark_Execution_Caching_and_Debugging.md>) *(Spark Core)*
- [Handling Poison Records](<../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Handling_Poison_Records.md>) *(Pipeline Orchestration)*

