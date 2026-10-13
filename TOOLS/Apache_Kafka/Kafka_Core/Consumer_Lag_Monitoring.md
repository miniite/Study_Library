---
id: "TOOLS/Apache_Kafka/Kafka_Core/Consumer_Lag_Monitoring"
level: 4
type: "concept"
topic: "TOOLS/Apache_Kafka/Kafka_Core"
tools: ["Apache_Kafka", "Prometheus", "Grafana"]
---

# Consumer Lag Monitoring

**L4 Advanced** · [↑ Kafka Core](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Apache Kafka](<../_index.md>)
- [Prometheus](<../../Prometheus/_index.md>)
- [Grafana](<../../Grafana/_index.md>)

## Practice questions

- [Basic] [What does 'consumer lag' mean for someone whose ETL job reads from Kafka and why should you watch it?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-31006_What_does_consumer_lag_mean_for_someone_whose_ETL_job_reads_.md>)
- [Intermediate] [How would you monitor UnderReplicatedPartitions and consumer lag on the cluster carrying your observability pipeline, and which is more urgent for anomaly detection?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-30941_How_would_you_monitor_UnderReplicatedPartitions_and_consumer.md>)
- [Intermediate] [How would you build alerting that distinguishes 'consumer lag is high' from 'data is genuinely stale' for a Kafka-fed warehouse, so on-call isn't paged on harmless catch-up lag?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-31153_How_would_you_build_alerting_that_distinguishes_consumer_lag.md>)
- [Deep] [An interviewer probe: every deploy triggers a rebalance and your lag spikes for a minute. What two things would you tune?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-30671_An_interviewer_probe_every_deploy_triggers_a_rebalance_and_y.md>)
- [Deep] [An interviewer asks: why might your anomaly consumer show zero lag yet operators complain alerts are late? How can lag mislead you in an observability pipeline?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-30915_An_interviewer_asks_why_might_your_anomaly_consumer_show_zer.md>)
- [Deep] [An interviewer asks: how would you detect a stuck anomaly consumer that holds a partition assignment but silently stopped scoring mid-poll?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-30958_An_interviewer_asks_how_would_you_detect_a_stuck_anomaly_con.md>)
- [Deep] [How would you isolate a slow, heavy analytics/backfill consumer so it never inflates the lag of the real-time warehouse loader reading the same events?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-31151_How_would_you_isolate_a_slow_heavy_analytics_backfill_consum.md>)
- [Expert] [An interviewer probe: lag is zero but users complain data is stale. How can lag be zero yet the pipeline be slow?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-30704_An_interviewer_probe_lag_is_zero_but_users_complain_data_is_.md>)

## Related concepts

- [Consumer Lag Diagnosis](<../../../03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Consumer_Lag_Diagnosis.md>) *(Big Data Processing)*
- [Partition Lag and Hot Keys](<../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Partition_Lag_and_Hot_Keys/_index.md>) *(Distributed Systems Core)*
- [Latency Budgets in Streaming Pipelines](<../../../03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Latency_Budgets_in_Streaming_Pipelines.md>) *(Big Data Processing)*

