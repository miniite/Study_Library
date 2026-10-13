---
id: "03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Event-time_Watermarks"
level: 5
type: "deep-dive"
topic: "03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture"
tools: ["Apache_Kafka"]
---

# Event-time Watermarks

**L5 Expert** · [↑ Streaming Architecture](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Apache Kafka](<../../../../TOOLS/Apache_Kafka/_index.md>)

## Practice questions

- [Intermediate] [Walk me through choosing tumbling versus sliding windows for a streaming rate-of-change detector on CPU utilization. Which gives smoother alerts and which reacts faster, and how do you pick?](<../../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Q-38143_Walk_me_through_choosing_tumbling_versus_sliding_windows_for.md>)
- [Deep] [Late-arriving log events routinely show up two minutes after their event-time in your observability pipeline. How do you set allowed-lateness and watermarks so your rolling error-rate window doesn't fire prematurely and page on-call for nothing?](<../../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Q-38129_Late-arriving_log_events_routinely_show_up_two_minutes_after.md>)
- [Deep] [Your metrics arrive with timestamps from thousands of hosts whose clocks drift by seconds. How does clock skew corrupt event-time windowing for anomaly detection, and when do you override host time with ingestion time?](<../../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Q-38157_Your_metrics_arrive_with_timestamps_from_thousands_of_hosts_.md>)
- [Deep] [You're asked to set the retention and tiered-storage policy for the org's telemetry log so incident replay works for weeks but cost stays bounded. What tiering do you standardize, and how do you handle the one compliance team that needs seven years?](<../../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Q-38245_You_re_asked_to_set_the_retention_and_tiered-storage_policy_.md>)
- [Expert] [Describe how you'd design a streaming pipeline to compute a rolling p99 latency SLI in real time. What algorithm keeps the percentile approximate but bounded in memory across high cardinality?](<../../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Q-38148_Describe_how_you_d_design_a_streaming_pipeline_to_compute_a_.md>)
- [Expert] [Walk me through a postmortem: a watermark stall in one operator silently held back windows across a shared cluster and caused hours of delayed alerts before anyone noticed. What's the root cause class, and what standing controls prevent this class org-wide?](<../../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Q-38038_Walk_me_through_a_postmortem_a_watermark_stall_in_one_operat.md>)
- [Expert] [Design event-time watermarking for a pipeline that ingests from mobile clients that can be offline for days. How do you balance allowed-lateness against unbounded state, and what product compromises do you negotiate?](<../../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Q-38052_Design_event-time_watermarking_for_a_pipeline_that_ingests_f.md>)
- [Expert] [Design a real-time alerting pipeline where a false negative (missed alert) is catastrophic but false positives erode trust. How do watermarks, allowed-lateness, and window firing policy interact with this asymmetry, and what defaults do you pick?](<../../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Q-38063_Design_a_real-time_alerting_pipeline_where_a_false_negative_.md>)

## Related concepts

- [Clock Skew and Time Windows](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Clock_Skew_and_Time_Windows/_index.md>) *(Distributed Systems Core)*
- [Real-time Counters](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Real-time_Counters.md>) *(High Level Design)*
- [Watermarks and State Management](<../../../Distributed_Processing/Batch_vs_Stream/Watermarks_and_State_Management.md>) *(Batch vs Stream)*

## References

- *Databricks Certified Associate Developer for Apache Spark* — Saba Shah — Structured Streaming concepts

