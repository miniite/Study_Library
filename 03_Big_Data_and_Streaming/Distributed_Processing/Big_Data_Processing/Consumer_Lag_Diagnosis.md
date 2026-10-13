---
id: "03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Consumer_Lag_Diagnosis"
level: 4
type: "concept"
topic: "03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing"
tools: ["Apache_Kafka", "Apache_Spark", "Prometheus"]
---

# Consumer Lag Diagnosis

**L4 Advanced** · [↑ Big Data Processing](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Apache Kafka](<../../../TOOLS/Apache_Kafka/_index.md>)
- [Apache Spark](<../../../TOOLS/Apache_Spark/_index.md>)
- [Prometheus](<../../../TOOLS/Prometheus/_index.md>)

## Practice questions

- [Intermediate] [A consumer is lagging behind on a busy telemetry topic; what basic steps would you take to find out why?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Q-04634_A_consumer_is_lagging_behind_on_a_busy_telemetry_topic_what_.md>)
- [Intermediate] [What basic metrics would you watch on a Kafka cluster to know it is healthy?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Q-04672_What_basic_metrics_would_you_watch_on_a_Kafka_cluster_to_kno.md>)
- [Deep] [You run a 300-partition telemetry topic and a rebalance storm keeps stalling ingestion during deploys; how would you diagnose and stabilize the consumer group?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Q-04696_You_run_a_300-partition_telemetry_topic_and_a_rebalance_stor.md>)
- [Deep] [Explain how you'd tune Kafka producer batching and linger for a high-throughput agent fleet without hurting alert latency on the critical path.](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Q-04708_Explain_how_you_d_tune_Kafka_producer_batching_and_linger_fo.md>)
- [Deep] [How would you design consumer isolation so that a buggy analytics job can't degrade the SLA of the paging pipeline sharing the same cluster?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Q-04771_How_would_you_design_consumer_isolation_so_that_a_buggy_anal.md>)
- [Deep] [You've been asked to cut the pipeline's p99 alert latency in half; walk me through where you'd look for the biggest wins and what you'd measure first.](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Q-04792_You_ve_been_asked_to_cut_the_pipeline_s_p99_alert_latency_in.md>)
- [Deep] [Producers on a critical topic occasionally throw TimeoutException under load; how do you decide between raising buffer.memory, max.block.ms, or scaling brokers?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Q-04920_Producers_on_a_critical_topic_occasionally_throw_TimeoutExce.md>)
- [Expert] [Your cross-datacenter Kafka replication (MirrorMaker) is lagging and alerts arrive out of order; how would you diagnose and mitigate?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Q-04729_Your_cross-datacenter_Kafka_replication_MirrorMaker_is_laggi.md>)

## Related concepts

- [Consumer Lag Monitoring](<../../../TOOLS/Apache_Kafka/Kafka_Core/Consumer_Lag_Monitoring.md>) *(Kafka Core)*
- [Pipeline Lag Monitoring](<../../../TOOLS/Apache_Kafka/Kafka_Core/Pipeline_Lag_Monitoring.md>) *(Kafka Core)*
- [Kafka Streams and Event Time](<../../../TOOLS/Apache_Kafka/Kafka_Core/Kafka_Streams_and_Event_Time/_index.md>) *(Kafka Core)*

