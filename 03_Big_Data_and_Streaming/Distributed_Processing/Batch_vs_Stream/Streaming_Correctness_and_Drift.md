---
id: "03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream/Streaming_Correctness_and_Drift"
level: 4
type: "concept"
topic: "03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream"
tools: ["Apache_Kafka", "Apache_Flink", "Prometheus"]
---

# Streaming Correctness and Drift

**L4 Advanced** · [↑ Batch vs Stream](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Apache Kafka](<../../../TOOLS/Apache_Kafka/_index.md>)
- [Apache Flink](<../../../TOOLS/Apache_Flink/_index.md>)
- [Prometheus](<../../../TOOLS/Prometheus/_index.md>)

## Practice questions

- [Intermediate] [What are late-arriving events in a stream and why are they a challenge?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream/Q-04213_What_are_late-arriving_events_in_a_stream_and_why_are_they_a.md>)
- [Intermediate] [How would you handle a sudden spike in incoming events in a streaming pipeline?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream/Q-04236_How_would_you_handle_a_sudden_spike_in_incoming_events_in_a_.md>)
- [Intermediate] [How do you monitor a streaming pipeline to know it is healthy and keeping up?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream/Q-04247_How_do_you_monitor_a_streaming_pipeline_to_know_it_is_health.md>)
- [Intermediate] [What is 'consumer lag' in Kafka and why is it an important metric?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream/Q-04248_What_is_consumer_lag_in_Kafka_and_why_is_it_an_important_met.md>)
- [Deep] [What does 'backpressure' mean in a streaming system?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream/Q-04235_What_does_backpressure_mean_in_a_streaming_system.md>)
- [Deep] [Your streaming job silently stopped advancing offsets but didn't crash. How would you detect and alert on a 'stuck but alive' consumer?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream/Q-04270_Your_streaming_job_silently_stopped_advancing_offsets_but_di.md>)
- [Deep] [Your batch pipeline's upstream file lands late one day and everything downstream cascades. How do you make the DAG resilient to late-arriving inputs?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream/Q-04275_Your_batch_pipeline_s_upstream_file_lands_late_one_day_and_e.md>)
- [Deep] [A streaming pipeline processes each event correctly but downstream aggregates drift over time. How do you detect and correct slow accumulating error?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream/Q-04278_A_streaming_pipeline_processes_each_event_correctly_but_down.md>)

## Related concepts

- [Latency Budgets in Streaming Pipelines](<../Big_Data_Processing/Latency_Budgets_in_Streaming_Pipelines.md>) *(Big Data Processing)*
- [Graceful Degradation in Streaming](<../../Streaming_and_Messaging/Streaming_Architecture/Provisional_and_Final_Results/Graceful_Degradation_in_Streaming.md>) *(Streaming Architecture)*
- [Provisional and Final Results](<../../Streaming_and_Messaging/Streaming_Architecture/Provisional_and_Final_Results/_index.md>) *(Streaming Architecture)*

