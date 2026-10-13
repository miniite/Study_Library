---
id: "TOOLS/Apache_Flink/Flink_Core/Exactly_Once_into_Kafka"
level: 4
type: "concept"
topic: "TOOLS/Apache_Flink/Flink_Core"
tools: ["Apache_Flink", "Apache_Kafka"]
---

# Exactly Once into Kafka

**L4 Advanced** · [↑ Flink Core](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Apache Flink](<../_index.md>)
- [Apache Kafka](<../../Apache_Kafka/_index.md>)

## Practice questions

- [Deep] [How do you guarantee ordering of remediation actions within a service key when Flink parallelizes processing across tasks?](<../../../PRACTICE/TOOLS/Apache_Flink/Flink_Core/Q-41996_How_do_you_guarantee_ordering_of_remediation_actions_within_.md>)
- [Deep] [A rebalance in Kafka causes duplicate events in your Flink pipeline. Is that a correctness problem, and how do you reason about it?](<../../../PRACTICE/TOOLS/Apache_Flink/Flink_Core/Q-42006_A_rebalance_in_Kafka_causes_duplicate_events_in_your_Flink_p.md>)
- [Deep] [When is at-least-once acceptable versus exactly-once for an alerting pipeline, and how do you make downstream idempotent instead?](<../../../PRACTICE/TOOLS/Apache_Flink/Flink_Core/Q-42026_When_is_at-least-once_acceptable_versus_exactly-once_for_an_.md>)
- [Deep] [How would you implement exactly-once delivery from Flink into Kafka for a transaction event stream, and what does the transactional sink cost you?](<../../../PRACTICE/TOOLS/Apache_Flink/Flink_Core/Q-42059_How_would_you_implement_exactly-once_delivery_from_Flink_int.md>)
- [Deep] [A Kafka rebalance causes duplicate events into your Flink pipeline. Is that a correctness problem for your warehouse counts, and how do you reason about it?](<../../../PRACTICE/TOOLS/Apache_Flink/Flink_Core/Q-42092_A_Kafka_rebalance_causes_duplicate_events_into_your_Flink_pi.md>)
- [Deep] [When is at-least-once acceptable versus exactly-once for a warehouse-load pipeline, and how do you make the sink idempotent instead?](<../../../PRACTICE/TOOLS/Apache_Flink/Flink_Core/Q-42107_When_is_at-least-once_acceptable_versus_exactly-once_for_a_w.md>)
- [Deep] [How would you compute a streaming running-total revenue per merchant that survives restarts and never double-counts?](<../../../PRACTICE/TOOLS/Apache_Flink/Flink_Core/Q-42108_How_would_you_compute_a_streaming_running-total_revenue_per_.md>)
- [Deep] [How do you guarantee per-key ordering of upserts to your warehouse when Flink parallelizes processing across subtasks?](<../../../PRACTICE/TOOLS/Apache_Flink/Flink_Core/Q-42127_How_do_you_guarantee_per-key_ordering_of_upserts_to_your_war.md>)

## Related concepts

- [Optimistic Concurrency and Clocks](<../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Joins_in_MPP_Systems/Optimistic_Concurrency_and_Clocks.md>) *(Distributed Systems Core)*
- [Batch and Stream Reconciliation](<../../../03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream/Batch_and_Stream_Reconciliation/_index.md>) *(Batch vs Stream)*
- [Exactly Once Spark Writes](<../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Exactly_Once_Spark_Writes.md>) *(Pipeline Orchestration)*

