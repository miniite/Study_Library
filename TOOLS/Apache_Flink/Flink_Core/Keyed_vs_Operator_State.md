---
id: "TOOLS/Apache_Flink/Flink_Core/Keyed_vs_Operator_State"
level: 4
type: "concept"
topic: "TOOLS/Apache_Flink/Flink_Core"
tools: ["Apache_Flink"]
---

# Keyed vs Operator State

**L4 Advanced** · [↑ Flink Core](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Apache Flink](<../_index.md>)

## Practice questions

- [Intermediate] [Explain how you'd expose custom Flink metrics per operator and wire them to Prometheus for the anomaly pipeline's own SLOs.](<../../../PRACTICE/TOOLS/Apache_Flink/Flink_Core/Q-41990_Explain_how_you_d_expose_custom_Flink_metrics_per_operator_a.md>)
- [Intermediate] [What's the difference between logical and physical operator IDs in the job graph and why does that matter when reading the Flink UI during an incident?](<../../../PRACTICE/TOOLS/Apache_Flink/Flink_Core/Q-42052_What_s_the_difference_between_logical_and_physical_operator_.md>)
- [Intermediate] [What's the difference between a KeyedProcessFunction and a windowed aggregate when you need custom, non-window-aligned emission logic?](<../../../PRACTICE/TOOLS/Apache_Flink/Flink_Core/Q-42144_What_s_the_difference_between_a_KeyedProcessFunction_and_a_w.md>)
- [Deep] [How would you implement alert deduplication and noise reduction using a keyed process function with state and timers in Flink?](<../../../PRACTICE/TOOLS/Apache_Flink/Flink_Core/Q-41959_How_would_you_implement_alert_deduplication_and_noise_reduct.md>)
- [Deep] [Why does serializer choice matter for Flink state, and what problems arise if you naively use Kryo for evolving telemetry POJOs?](<../../../PRACTICE/TOOLS/Apache_Flink/Flink_Core/Q-41993_Why_does_serializer_choice_matter_for_Flink_state_and_what_p.md>)
- [Deep] [When would you use operator chaining versus disabling it, and how does that affect debugging a slow alerting stage?](<../../../PRACTICE/TOOLS/Apache_Flink/Flink_Core/Q-42042_When_would_you_use_operator_chaining_versus_disabling_it_and.md>)
- [Deep] [Why does serializer choice matter for Flink state, and what breaks if you rely on Kryo for an evolving event POJO?](<../../../PRACTICE/TOOLS/Apache_Flink/Flink_Core/Q-42080_Why_does_serializer_choice_matter_for_Flink_state_and_what_b.md>)
- [Deep] [How does keyBy affect network shuffle, and where is that cost visible in a high-cardinality streaming aggregation?](<../../../PRACTICE/TOOLS/Apache_Flink/Flink_Core/Q-42117_How_does_keyBy_affect_network_shuffle_and_where_is_that_cost.md>)

## Related concepts

- [Fat vs Thin Events](<../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Fat_vs_Thin_Events/_index.md>) *(Event Driven Architecture)*
- [HttpClient and Resilient Outbound Calls](<../../DotNet/DotNet_Platform/HttpClient_and_Resilient_Outbound_Calls.md>) *(DotNet Platform)*
- [Idempotent Consumer Design](<../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Idempotent_Consumer_Design/_index.md>) *(Low Level Design)*

