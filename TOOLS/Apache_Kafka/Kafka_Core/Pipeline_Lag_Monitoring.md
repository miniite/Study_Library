---
id: "TOOLS/Apache_Kafka/Kafka_Core/Pipeline_Lag_Monitoring"
level: 4
type: "concept"
topic: "TOOLS/Apache_Kafka/Kafka_Core"
tools: ["Apache_Kafka", "Prometheus"]
---

# Pipeline Lag Monitoring

**L4 Advanced** · [↑ Kafka Core](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Apache Kafka](<../_index.md>)
- [Prometheus](<../../Prometheus/_index.md>)

## Practice questions

- [Basic] [How would you use kafka-console-producer and kafka-console-consumer to quickly debug whether messages are flowing?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-30656_How_would_you_use_kafka-console-producer_and_kafka-console-c.md>)
- [Intermediate] [How does the producer's callback and Future help you know if a send succeeded?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-30599_How_does_the_producer_s_callback_and_Future_help_you_know_if.md>)
- [Intermediate] [Your producer says a send failed with a timeout. As a data engineer feeding Kafka from a batch job what basic checks would you do?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-31029_Your_producer_says_a_send_failed_with_a_timeout_As_a_data_en.md>)
- [Intermediate] [You inherit a pipeline with no monitoring. What are the first three things you would add to know it is healthy?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-31061_You_inherit_a_pipeline_with_no_monitoring_What_are_the_first.md>)
- [Intermediate] [Your bulk export producer hits buffer.memory exhaustion and starts blocking; how do you detect this before it silently stalls the pipeline and what do you tune?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-31169_Your_bulk_export_producer_hits_buffer_memory_exhaustion_and_.md>)
- [Deep] [An interviewer asks: your consumer processes each record with a 30-second external call and you keep getting kicked out of the group. What single setting is likely misconfigured?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-30616_An_interviewer_asks_your_consumer_processes_each_record_with.md>)
- [Deep] [Your metrics producers batch aggressively but a critical health-check event sits waiting for the batch to fill. How do you serve both bulk telemetry and urgent signals through one producer?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-30957_Your_metrics_producers_batch_aggressively_but_a_critical_hea.md>)
- [Deep] [How would you throttle a single misbehaving client that is saturating broker network and causing latency for everyone else, without redeploying that client?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-30768_How_would_you_throttle_a_single_misbehaving_client_that_is_s.md>)

## Related concepts

- [Consumer Lag Diagnosis](<../../../03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Consumer_Lag_Diagnosis.md>) *(Big Data Processing)*
- [Tiered Storage for Logs](<../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Tiered_Storage_for_Logs.md>) *(Event Driven Architecture)*
- [Consuming Untrusted Streams](<../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Provisional_and_Final_Results/Consuming_Untrusted_Streams.md>) *(Streaming Architecture)*

