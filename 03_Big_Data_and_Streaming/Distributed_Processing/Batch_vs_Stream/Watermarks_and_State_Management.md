---
id: "03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream/Watermarks_and_State_Management"
level: 4
type: "concept"
topic: "03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream"
tools: []
---

# Watermarks and State Management

**L4 Advanced** · [↑ Batch vs Stream](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Practice questions

- [Deep] [What is a watermark in stream processing and what problem does it solve?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream/Q-04214_What_is_a_watermark_in_stream_processing_and_what_problem_do.md>)
- [Deep] [An interviewer asks how you'd guarantee a windowed count is final so downstream can act on it. How do watermarks and allowed lateness give you that?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream/Q-04272_An_interviewer_asks_how_you_d_guarantee_a_windowed_count_is_.md>)
- [Deep] [How do you handle the case where your streaming source (Kafka) retention expired before your consumer could reprocess after a long outage?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream/Q-04283_How_do_you_handle_the_case_where_your_streaming_source_Kafka.md>)
- [Deep] [How do you handle a streaming aggregation that needs to emit results per day boundary in a specific timezone across DST changes?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream/Q-04289_How_do_you_handle_a_streaming_aggregation_that_needs_to_emit.md>)
- [Deep] [A streaming aggregation's state store grows unbounded over weeks. How do you decide the right watermark and state TTL without dropping legitimate late events?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream/Q-04326_A_streaming_aggregation_s_state_store_grows_unbounded_over_w.md>)
- [Deep] [A stream-stream join is silently dropping matches. How do you reason about the join watermark bounds and buffered state to find the cause?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream/Q-04335_A_stream-stream_join_is_silently_dropping_matches_How_do_you.md>)
- [Deep] [How do you tune watermark delay so you're not trading too much latency for correctness on late data? What data do you look at to set it?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream/Q-04366_How_do_you_tune_watermark_delay_so_you_re_not_trading_too_mu.md>)
- [Deep] [How do you decide the compaction and retention policy on a Kafka topic that feeds both a streaming consumer and periodic batch reloads?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream/Q-04371_How_do_you_decide_the_compaction_and_retention_policy_on_a_K.md>)

## Related concepts

- [Event-time Watermarks](<../../Streaming_and_Messaging/Streaming_Architecture/Provisional_and_Final_Results/Event-time_Watermarks.md>) *(Streaming Architecture)*
- [End to End No Data Loss](<../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/End_to_End_No_Data_Loss.md>) *(Data Architecture)*
- [Graceful Degradation in Streaming](<../../Streaming_and_Messaging/Streaming_Architecture/Provisional_and_Final_Results/Graceful_Degradation_in_Streaming.md>) *(Streaming Architecture)*

