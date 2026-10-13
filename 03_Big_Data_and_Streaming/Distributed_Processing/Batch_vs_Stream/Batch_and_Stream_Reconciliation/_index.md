---
id: "03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream/Batch_and_Stream_Reconciliation"
level: 4
type: "concept"
topic: "03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream"
tools: ["Apache_Kafka", "Apache_Flink"]
---

# Batch and Stream Reconciliation

**L4 Advanced** · [↑ Batch vs Stream](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Go deeper

- [Unified Batch and Streaming Platforms](<Unified_Batch_and_Streaming_Platforms.md>)
- [Unified Batch and Streaming Semantics](<Unified_Batch_and_Streaming_Semantics.md>)

## Implementations (tools)

- [Apache Kafka](<../../../../TOOLS/Apache_Kafka/_index.md>)
- [Apache Flink](<../../../../TOOLS/Apache_Flink/_index.md>)

## Practice questions

- [Intermediate] [If a streaming job goes down for an hour, what happens to the data during that gap and how could you recover it?](<../../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream/Q-04208_If_a_streaming_job_goes_down_for_an_hour_what_happens_to_the.md>)
- [Deep] [What is idempotency and why does it matter for both batch and streaming pipelines?](<../../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream/Q-04229_What_is_idempotency_and_why_does_it_matter_for_both_batch_an.md>)
- [Deep] [How do you ensure a streaming job does not lose data when it restarts after a crash?](<../../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream/Q-04320_How_do_you_ensure_a_streaming_job_does_not_lose_data_when_it.md>)
- [Deep] [How would you validate that switching a critical pipeline from batch to streaming didn't change any numbers before you cut over production traffic?](<../../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream/Q-04292_How_would_you_validate_that_switching_a_critical_pipeline_fr.md>)
- [Deep] [A CDC-based streaming load falls behind whenever the source runs a bulk update. How do you keep the consumer from being overwhelmed by that burst?](<../../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream/Q-04339_A_CDC-based_streaming_load_falls_behind_whenever_the_source_.md>)
- [Deep] [How do you test a streaming pipeline for correctness given you can't just re-run it deterministically like a batch job?](<../../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream/Q-04358_How_do_you_test_a_streaming_pipeline_for_correctness_given_y.md>)
- [Deep] [You're seeing duplicate events end-to-end despite Kafka exactly-once being enabled. Where beyond Kafka could duplication be introduced?](<../../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream/Q-04370_You_re_seeing_duplicate_events_end-to-end_despite_Kafka_exac.md>)
- [Deep] [How do you decide the checkpoint interval trade-off: too frequent hurts throughput, too rare costs reprocessing on failure?](<../../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream/Q-04394_How_do_you_decide_the_checkpoint_interval_trade-off_too_freq.md>)

## Related concepts

- [Unified Batch and Streaming Semantics](<Unified_Batch_and_Streaming_Semantics.md>) *(Batch vs Stream)*
- [Provisional and Final Results](<../../../Streaming_and_Messaging/Streaming_Architecture/Provisional_and_Final_Results/_index.md>) *(Streaming Architecture)*
- [Optimistic Concurrency and Clocks](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Joins_in_MPP_Systems/Optimistic_Concurrency_and_Clocks.md>) *(Distributed Systems Core)*

