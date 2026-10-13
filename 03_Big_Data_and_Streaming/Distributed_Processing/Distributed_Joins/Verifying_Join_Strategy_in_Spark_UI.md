---
id: "03_Big_Data_and_Streaming/Distributed_Processing/Distributed_Joins/Verifying_Join_Strategy_in_Spark_UI"
level: 4
type: "concept"
topic: "03_Big_Data_and_Streaming/Distributed_Processing/Distributed_Joins"
tools: ["Apache_Spark"]
---

# Verifying Join Strategy in Spark UI

**L4 Advanced** · [↑ Distributed Joins](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Apache Spark](<../../../TOOLS/Apache_Spark/_index.md>)

## Practice questions

- [Intermediate] [Describe how you'd read the Spark UI to confirm whether a join actually broadcast or fell back to shuffle.](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Distributed_Joins/Q-05208_Describe_how_you_d_read_the_Spark_UI_to_confirm_whether_a_jo.md>)
- [Intermediate] [What is spark.sql.autoBroadcastJoinThreshold and what pitfalls come from setting it too high in production?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Distributed_Joins/Q-05229_What_is_spark_sql_autoBroadcastJoinThreshold_and_what_pitfal.md>)
- [Intermediate] [How do you set spark.sql.autoBroadcastJoinThreshold to -1 safely and what's the blast radius of that change?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Distributed_Joins/Q-05288_How_do_you_set_spark_sql_autoBroadcastJoinThreshold_to_-1_sa.md>)
- [Deep] [You see a BroadcastNestedLoopJoin in the physical plan — what caused it and why is it dangerous?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Distributed_Joins/Q-05204_You_see_a_BroadcastNestedLoopJoin_in_the_physical_plan_what_.md>)
- [Deep] [Describe a production incident where a broadcast join caused a driver OOM and how you resolved it.](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Distributed_Joins/Q-05225_Describe_a_production_incident_where_a_broadcast_join_caused.md>)
- [Deep] [How do you detect that AQE disabled a broadcast because it read stale or missing statistics?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Distributed_Joins/Q-05238_How_do_you_detect_that_AQE_disabled_a_broadcast_because_it_r.md>)
- [Deep] [How do you tell from executor logs that a broadcast is being re-serialized for every task rather than once per executor?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Distributed_Joins/Q-05259_How_do_you_tell_from_executor_logs_that_a_broadcast_is_being.md>)
- [Deep] [What happens if you disable AQE entirely — how does that change the risk profile of your join-heavy pipeline?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Distributed_Joins/Q-05265_What_happens_if_you_disable_AQE_entirely_how_does_that_chang.md>)

## Related concepts

- [Shuffle Tuning and Broadcast Joins](<../../../TOOLS/Apache_Spark/Spark_Core/Shuffle_Tuning_and_Broadcast_Joins.md>) *(Spark Core)*
- [Spark Skew and Spill](<../../../TOOLS/Apache_Spark/Spark_Optimization/Spark_Skew_and_Spill.md>) *(Spark Optimization)*
- [Bucketed Joins in Spark](<../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Bucketed_Joins_in_Spark.md>) *(Pipeline Orchestration)*

