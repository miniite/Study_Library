---
id: "TOOLS/Apache_Flink/Flink_Core/Checkpoint_Overhead"
level: 4
type: "concept"
topic: "TOOLS/Apache_Flink/Flink_Core"
tools: ["Apache_Flink"]
---

# Checkpoint Overhead

**L4 Advanced** · [↑ Flink Core](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Apache Flink](<../_index.md>)

## Practice questions

- [Intermediate] [Which restart strategy (fixed-delay, exponential-backoff, failure-rate) would you pick for a critical ingestion job and why?](<../../../PRACTICE/TOOLS/Apache_Flink/Flink_Core/Q-42115_Which_restart_strategy_fixed-delay_exponential-backoff_failu.md>)
- [Deep] [How does backpressure propagate through a Flink job graph, and what metrics do you watch to localize the bottleneck operator?](<../../../PRACTICE/TOOLS/Apache_Flink/Flink_Core/Q-41968_How_does_backpressure_propagate_through_a_Flink_job_graph_an.md>)
- [Deep] [How would you throttle or rate-limit automated remediation actions emitted by a Flink pipeline to prevent an automation-driven outage?](<../../../PRACTICE/TOOLS/Apache_Flink/Flink_Core/Q-42014_How_would_you_throttle_or_rate-limit_automated_remediation_a.md>)
- [Deep] [How do you provision resources (task slots, managed memory, CPU) for a Flink job so it survives incident-time traffic surges?](<../../../PRACTICE/TOOLS/Apache_Flink/Flink_Core/Q-42033_How_do_you_provision_resources_task_slots_managed_memory_CPU.md>)
- [Deep] [How do you cost-optimize a 24x7 Flink alerting pipeline that is over-provisioned for peak but idle most of the time?](<../../../PRACTICE/TOOLS/Apache_Flink/Flink_Core/Q-42045_How_do_you_cost-optimize_a_24x7_Flink_alerting_pipeline_that.md>)
- [Deep] [A downstream JDBC sink to Postgres throttles your Flink job and lag climbs. How do you localize and fix the backpressure?](<../../../PRACTICE/TOOLS/Apache_Flink/Flink_Core/Q-42066_A_downstream_JDBC_sink_to_Postgres_throttles_your_Flink_job_.md>)
- [Deep] [Throughput dropped after you enabled exactly-once. How do you quantify and then reduce the checkpointing overhead?](<../../../PRACTICE/TOOLS/Apache_Flink/Flink_Core/Q-42106_Throughput_dropped_after_you_enabled_exactly-once_How_do_you.md>)
- [Deep] [How do you cost-optimize a 24x7 Flink job that is over-provisioned for peak but mostly idle overnight?](<../../../PRACTICE/TOOLS/Apache_Flink/Flink_Core/Q-42121_How_do_you_cost-optimize_a_24x7_Flink_job_that_is_over-provi.md>)

## Related concepts

- [Graceful Degradation in Streaming](<../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Provisional_and_Final_Results/Graceful_Degradation_in_Streaming.md>) *(Streaming Architecture)*
- [Org-wide Backpressure](<../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Rate_Limiter_Storage/Org-wide_Backpressure.md>) *(System Design Core)*
- [Slow Dependencies and Load Shedding](<../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Deadline_Propagation/Slow_Dependencies_and_Load_Shedding.md>) *(Distributed Systems Core)*

