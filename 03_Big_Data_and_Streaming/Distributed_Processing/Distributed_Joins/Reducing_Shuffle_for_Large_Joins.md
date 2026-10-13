---
id: "03_Big_Data_and_Streaming/Distributed_Processing/Distributed_Joins/Reducing_Shuffle_for_Large_Joins"
level: 4
type: "concept"
topic: "03_Big_Data_and_Streaming/Distributed_Processing/Distributed_Joins"
tools: ["Apache_Spark"]
---

# Reducing Shuffle for Large Joins

**L4 Advanced** · [↑ Distributed Joins](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Apache Spark](<../../../TOOLS/Apache_Spark/_index.md>)

## Practice questions

- [Intermediate] [Describe how you would use a broadcast join hint and when a hint gets silently ignored by the optimizer.](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Distributed_Joins/Q-05198_Describe_how_you_would_use_a_broadcast_join_hint_and_when_a_.md>)
- [Intermediate] [How do you decide whether to cache and broadcast a reference table or re-broadcast it per query?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Distributed_Joins/Q-05230_How_do_you_decide_whether_to_cache_and_broadcast_a_reference.md>)
- [Intermediate] [How would you explain the broadcast-versus-shuffle trade-off to a junior engineer using a concrete cost model?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Distributed_Joins/Q-05279_How_would_you_explain_the_broadcast-versus-shuffle_trade-off.md>)
- [Deep] [Explain why a broadcast join eliminates the exchange on the large side but not necessarily the scan cost.](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Distributed_Joins/Q-05213_Explain_why_a_broadcast_join_eliminates_the_exchange_on_the_.md>)
- [Deep] [A shuffle join produces one giant task that runs for hours — what's happening and how do you fix the partitioning?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Distributed_Joins/Q-05231_A_shuffle_join_produces_one_giant_task_that_runs_for_hours_w.md>)
- [Deep] [A pipeline shuffles the same dimension into 20 different jobs — how would you eliminate that repeated shuffle cost?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Distributed_Joins/Q-05244_A_pipeline_shuffles_the_same_dimension_into_20_different_job.md>)
- [Deep] [Explain how the broadcast variable is distributed to executors and why the driver can become a bottleneck.](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Distributed_Joins/Q-05262_Explain_how_the_broadcast_variable_is_distributed_to_executo.md>)
- [Deep] [How do you reason about broadcast versus shuffle when the cluster is autoscaling and node count is unpredictable?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Distributed_Joins/Q-05276_How_do_you_reason_about_broadcast_versus_shuffle_when_the_cl.md>)

## Related concepts

- [Spark Skew and Spill](<../../../TOOLS/Apache_Spark/Spark_Optimization/Spark_Skew_and_Spill.md>) *(Spark Optimization)*
- [Bucketed Joins in Spark](<../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Bucketed_Joins_in_Spark.md>) *(Pipeline Orchestration)*
- [Distributed Joins in MPP Systems](<../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Joins_in_MPP_Systems/_index.md>) *(Distributed Systems Core)*

