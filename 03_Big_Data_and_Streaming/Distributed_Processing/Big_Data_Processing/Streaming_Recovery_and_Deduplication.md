---
id: "03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Streaming_Recovery_and_Deduplication"
level: 4
type: "concept"
topic: "03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing"
tools: ["Apache_Spark", "Apache_Kafka"]
---

# Streaming Recovery and Deduplication

**L4 Advanced** · [↑ Big Data Processing](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Apache Spark](<../../../TOOLS/Apache_Spark/_index.md>)
- [Apache Kafka](<../../../TOOLS/Apache_Kafka/_index.md>)

## Practice questions

- [Intermediate] [Your Spark streaming job crashes and restarts; how do you avoid double counting the same events?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Q-04637_Your_Spark_streaming_job_crashes_and_restarts_how_do_you_avo.md>)
- [Intermediate] [How would you replay old events from Kafka into a Spark job after fixing a bug and why is that possible?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Q-04851_How_would_you_replay_old_events_from_Kafka_into_a_Spark_job_.md>)
- [Deep] [Your Spark streaming job restarts after a crash; how do you avoid processing the same events twice?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Q-04865_Your_Spark_streaming_job_restarts_after_a_crash_how_do_you_a.md>)
- [Deep] [How would you implement alert suppression/dedup windows in Spark so a flapping service doesn't generate 500 pages, without hiding a genuine second incident?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Q-04755_How_would_you_implement_alert_suppression_dedup_windows_in_S.md>)
- [Deep] [A CDC pipeline via Debezium and Kafka is duplicating updates in the warehouse after a connector restart; where do the dupes come from and how do you make the sink tolerant?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Q-04910_A_CDC_pipeline_via_Debezium_and_Kafka_is_duplicating_updates.md>)
- [Deep] [A poison-pill message crashes and restarts your streaming job in a loop; how do you make the job skip-and-record it without manual intervention?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Q-04964_A_poison-pill_message_crashes_and_restarts_your_streaming_jo.md>)
- [Expert] [Your checkpoint directory on the streaming job grew corrupt after an unclean shutdown; how do you recover without double-counting alerts or losing state?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Q-04707_Your_checkpoint_directory_on_the_streaming_job_grew_corrupt_.md>)
- [Expert] [Your Structured Streaming checkpoint got tied to a specific query plan; how do you evolve the job's logic without invalidating the checkpoint and losing offsets?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Q-04931_Your_Structured_Streaming_checkpoint_got_tied_to_a_specific_.md>)

## Related concepts

- [Handling Poison Records](<../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Handling_Poison_Records.md>) *(Pipeline Orchestration)*
- [Exactly Once Spark Writes](<../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Exactly_Once_Spark_Writes.md>) *(Pipeline Orchestration)*
- [Spark Execution Caching and Debugging](<../../../TOOLS/Apache_Spark/Spark_Core/Spark_Execution_Caching_and_Debugging.md>) *(Spark Core)*

