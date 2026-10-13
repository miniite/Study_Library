---
id: "03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Processing_Time_vs_Event_Time"
level: 5
type: "deep-dive"
topic: "03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture"
tools: ["Apache_Kafka"]
---

# Processing Time vs Event Time

**L5 Expert** · [↑ Streaming Architecture](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Apache Kafka](<../../../../TOOLS/Apache_Kafka/_index.md>)

## Practice questions

- [Deep] [Your alert-correlation engine consumes from a Kafka topic of raw events. Why would you key that topic by service or resource rather than by alert-id, and what breaks in correlation if you get the key wrong?](<../../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Q-38128_Your_alert-correlation_engine_consumes_from_a_Kafka_topic_of.md>)
- [Deep] [Explain the difference between at-least-once and exactly-once delivery for an alert stream, and why for correlation you might actually prefer at-least-once with downstream dedup. When does that reasoning fail?](<../../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Q-38151_Explain_the_difference_between_at-least-once_and_exactly-onc.md>)
- [Deep] [Design windowing semantics (tumbling, sliding, session) as a self-service abstraction your platform exposes so non-expert teams don't misuse them. Where do naive users most often get burned, and how does your API design prevent it?](<../../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Q-38039_Design_windowing_semantics_tumbling_sliding_session_as_a_sel.md>)
- [Deep] [An interviewer asks: 'Why event-driven streaming instead of periodic polling for cross-service communication?' Give the architectural trade-off at org scale, and then the cases where polling is the correct, boring, right answer.](<../../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Q-38114_An_interviewer_asks_Why_event-driven_streaming_instead_of_pe.md>)
- [Deep] [How do you decide the org-level boundary between what runs as a real-time stream processor and what runs as a periodic query against a time-series store? Give the decision framework you'd publish so hundreds of engineers make consistent choices.](<../../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Q-38240_How_do_you_decide_the_org-level_boundary_between_what_runs_a.md>)
- [Deep] [You're setting the org standard for how much aggregation happens at the edge collector versus in the central stream. What's the trade-off between edge pre-aggregation (cheap, lossy) and central raw ingestion (expensive, flexible), and how do you let teams tune it safely?](<../../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Q-38256_You_re_setting_the_org_standard_for_how_much_aggregation_hap.md>)
- [Deep] [How do you set the platform's stance on stateful versus stateless AIOps computations: which detections must hold state in the stream (baselines, sessions) versus be recomputed statelessly? What does pushing statefulness into the platform cost you operationally?](<../../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Q-38287_How_do_you_set_the_platform_s_stance_on_stateful_versus_stat.md>)
- [Expert] [Adjudicate the trade-off for us: when do you accept processing-time semantics over event-time correctness in a business-critical pipeline, and how do you make that decision defensible to a bar-raiser who claims you're cutting corners?](<../../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Q-38028_Adjudicate_the_trade-off_for_us_when_do_you_accept_processin.md>)

## Related concepts

- [Batch vs Streaming Cost Trade-offs](<../../../Distributed_Processing/Batch_vs_Stream/Batch_vs_Streaming_Cost_Trade-offs.md>) *(Batch vs Stream)*
- [Batch vs Streaming Features](<../../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Batch_vs_Streaming_Features/_index.md>) *(Pipeline Orchestration)*
- [Streaming-first Data Architecture](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/Streaming-first_Data_Architecture.md>) *(Data Architecture)*

## References

- *Azure Data Fundamentals* — Michael John Pena — Real-Time Analytics

