---
id: "03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Graceful_Degradation_in_Streaming"
level: 5
type: "deep-dive"
topic: "03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture"
tools: ["Apache_Kafka", "Apache_Flink", "AWS"]
---

# Graceful Degradation in Streaming

**L5 Expert** · [↑ Streaming Architecture](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Apache Kafka](<../../../../TOOLS/Apache_Kafka/_index.md>)
- [Apache Flink](<../../../../TOOLS/Apache_Flink/_index.md>)
- [AWS](<../../../../TOOLS/AWS/_index.md>)

## Practice questions

- [Intermediate] [Your Flink job computing streaming baselines for anomaly detection keeps falling behind during traffic spikes. How would you diagnose whether it's backpressure, checkpointing, or state growth, and what's your first mitigation?](<../../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Q-38131_Your_Flink_job_computing_streaming_baselines_for_anomaly_det.md>)
- [Deep] [Your streaming pipeline computes rolling baselines that must survive a job restart to avoid a cold-start of false positives. How do you persist and restore that state, and how big can it get?](<../../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Q-38163_Your_streaming_pipeline_computes_rolling_baselines_that_must.md>)
- [Deep] [Explain how you'd bound the memory of a stream-stream join between deploy events and error spikes when some deploys never produce a matching spike. What eviction policy avoids unbounded state?](<../../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Q-38216_Explain_how_you_d_bound_the_memory_of_a_stream-stream_join_b.md>)
- [Expert] [Your Flink state backend (RocksDB) is causing high disk I/O and slow checkpoints for a large-cardinality anomaly-baseline job. Walk me through tuning options and when you'd shard the job instead.](<../../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Q-38176_Your_Flink_state_backend_RocksDB_is_causing_high_disk_I_O_an.md>)
- [Expert] [An engineer proposes reprocessing a year of history by resetting consumer offsets to earliest on the live pipeline. Explain everything wrong with that at scale and design the safe reprocessing pattern you'd standardize instead.](<../../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Q-38050_An_engineer_proposes_reprocessing_a_year_of_history_by_reset.md>)
- [Expert] [Design tiered storage and retention strategy for a streaming backbone that must serve both real-time consumers and multi-year replay for compliance. How do you keep costs sane while making arbitrary historical reprocessing possible?](<../../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Q-38066_Design_tiered_storage_and_retention_strategy_for_a_streaming.md>)
- [Expert] [Walk me through how you'd bound and shard unbounded keyed state (e.g., per-user session state) when a small fraction of keys hold most of the data. What's your hot-key mitigation and state-TTL strategy, and how do you avoid unbounded memory growth?](<../../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Q-38082_Walk_me_through_how_you_d_bound_and_shard_unbounded_keyed_st.md>)
- [Expert] [How do you handle the operational reality that a streaming aggregate over a long window (say 30-day) makes state huge and reprocessing painful? Design an incremental/compacted approach and its trade-offs versus recomputing from raw.](<../../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Q-38118_How_do_you_handle_the_operational_reality_that_a_streaming_a.md>)

## Related concepts

- [Pipeline Design Patterns](<../../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Design/Pipeline_Design_Patterns.md>) *(Pipeline Design)*
- [Slow Dependencies and Load Shedding](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Deadline_Propagation/Slow_Dependencies_and_Load_Shedding.md>) *(Distributed Systems Core)*
- [Unified Batch and Streaming Semantics](<../../../Distributed_Processing/Batch_vs_Stream/Batch_and_Stream_Reconciliation/Unified_Batch_and_Streaming_Semantics.md>) *(Batch vs Stream)*

