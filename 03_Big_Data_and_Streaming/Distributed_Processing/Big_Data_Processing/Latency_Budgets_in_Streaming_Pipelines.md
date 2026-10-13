---
id: "03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Latency_Budgets_in_Streaming_Pipelines"
level: 4
type: "concept"
topic: "03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing"
tools: ["Apache_Spark", "Apache_Kafka"]
---

# Latency Budgets in Streaming Pipelines

**L4 Advanced** · [↑ Big Data Processing](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Apache Spark](<../../../TOOLS/Apache_Spark/_index.md>)
- [Apache Kafka](<../../../TOOLS/Apache_Kafka/_index.md>)

## Practice questions

- [Basic] [Would you use batch or streaming to compute a rolling five-minute error rate for alerting and why?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Q-04607_Would_you_use_batch_or_streaming_to_compute_a_rolling_five-m.md>)
- [Intermediate] [How would you compute the ninety-fifth percentile latency per endpoint from a stream of request logs?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Q-04649_How_would_you_compute_the_ninety-fifth_percentile_latency_pe.md>)
- [Intermediate] [How would you handle a burst of traffic that suddenly triples the event rate into your Kafka topic?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Q-04677_How_would_you_handle_a_burst_of_traffic_that_suddenly_triple.md>)
- [Deep] [How would you reason about the end-to-end latency from an event happening to an alert firing through Kafka and Spark?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Q-04687_How_would_you_reason_about_the_end-to-end_latency_from_an_ev.md>)
- [Deep] [An SRE complains alerts fire 90 seconds after incidents start; how would you trace and cut end-to-end latency across the Kafka+Spark chain?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Q-04710_An_SRE_complains_alerts_fire_90_seconds_after_incidents_star.md>)
- [Deep] [How would you enforce and monitor data quality (nulls, bad timestamps, cardinality explosions) inside the streaming path before it poisons downstream models?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Q-04748_How_would_you_enforce_and_monitor_data_quality_nulls_bad_tim.md>)
- [Deep] [How would you correlate deploy events from one topic with error spikes from another in a streaming join, accounting for the delay between deploy and impact?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Q-04765_How_would_you_correlate_deploy_events_from_one_topic_with_er.md>)
- [Deep] [How would you design event-time windowing so a delayed upstream batch of events doesn't corrupt already-emitted window results?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Q-04968_How_would_you_design_event-time_windowing_so_a_delayed_upstr.md>)

## Related concepts

- [Kafka Streams and Event Time](<../../../TOOLS/Apache_Kafka/Kafka_Core/Kafka_Streams_and_Event_Time/_index.md>) *(Kafka Core)*
- [Batch vs Streaming Features](<../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Batch_vs_Streaming_Features/_index.md>) *(Pipeline Orchestration)*
- [Streaming Correctness and Drift](<../Batch_vs_Stream/Streaming_Correctness_and_Drift.md>) *(Batch vs Stream)*

