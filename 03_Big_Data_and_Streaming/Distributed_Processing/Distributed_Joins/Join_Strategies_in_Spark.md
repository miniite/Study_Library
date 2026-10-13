---
id: "03_Big_Data_and_Streaming/Distributed_Processing/Distributed_Joins/Join_Strategies_in_Spark"
level: 4
type: "concept"
topic: "03_Big_Data_and_Streaming/Distributed_Processing/Distributed_Joins"
tools: ["Apache_Spark"]
---

# Join Strategies in Spark

**L4 Advanced** · [↑ Distributed Joins](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Apache Spark](<../../../TOOLS/Apache_Spark/_index.md>)

## Practice questions

- [Intermediate] [What signals in the DAG or metrics tell you a shuffle join is your pipeline's bottleneck?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Distributed_Joins/Q-05222_What_signals_in_the_DAG_or_metrics_tell_you_a_shuffle_join_i.md>)
- [Intermediate] [How would you tune spark.sql.shuffle.partitions differently for a 100 GB join versus a 5 TB join?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Distributed_Joins/Q-05233_How_would_you_tune_spark_sql_shuffle_partitions_differently_.md>)
- [Intermediate] [How do you inspect the Catalyst physical plan to see exactly which exchange and join operators will run?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Distributed_Joins/Q-05252_How_do_you_inspect_the_Catalyst_physical_plan_to_see_exactly.md>)
- [Deep] [A shuffle join is spilling terabytes to disk — how do you tell whether the problem is skew, partition count, or memory?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Distributed_Joins/Q-05199_A_shuffle_join_is_spilling_terabytes_to_disk_how_do_you_tell.md>)
- [Deep] [What is the effect of AQE coalescing shuffle partitions after a join and how does it interact with join skew handling?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Distributed_Joins/Q-05209_What_is_the_effect_of_AQE_coalescing_shuffle_partitions_afte.md>)
- [Deep] [Two large tables must be joined repeatedly — how do bucketing and pre-partitioning let you avoid the shuffle entirely?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Distributed_Joins/Q-05210_Two_large_tables_must_be_joined_repeatedly_how_do_bucketing_.md>)
- [Deep] [What's the difference between a shuffle-hash join and a sort-merge join and when does Spark pick each?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Distributed_Joins/Q-05249_What_s_the_difference_between_a_shuffle-hash_join_and_a_sort.md>)
- [Deep] [Why did Spark move away from shuffle-hash join as the default toward sort-merge, and how does that affect your tuning?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Distributed_Joins/Q-05250_Why_did_Spark_move_away_from_shuffle-hash_join_as_the_defaul.md>)

## Related concepts

- [Spark Skew and Spill](<../../../TOOLS/Apache_Spark/Spark_Optimization/Spark_Skew_and_Spill.md>) *(Spark Optimization)*
- [Bucketed Joins in Spark](<../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Bucketed_Joins_in_Spark.md>) *(Pipeline Orchestration)*

