---
id: "07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Tracing"
level: 5
type: "deep-dive"
topic: "07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core"
tools: []
---

# Distributed Tracing

**L5 Expert** · [↑ Distributed Systems Core](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Practice questions

- [Intermediate] [Your agent framework logs each step to a distributed trace, but tool sub-calls in a fan-out lose their parent context so traces are broken and you can't find the slow hop. Explain how context propagation should work across async tool calls and the sampling trade-off at high QPS.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23203_Your_agent_framework_logs_each_step_to_a_distributed_trace_b.md>)
- [Intermediate] [Your log pipeline via Fluent Bit to a distributed log store started dropping records during an incident spike, exactly when you needed the logs most. Walk me through where buffering versus dropping should live and how you avoid the pipeline masking the very incident it feeds detection.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23865_Your_log_pipeline_via_Fluent_Bit_to_a_distributed_log_store_.md>)
- [Deep] [Explain how distributed tracing (trace IDs, spans, context propagation) works and how you'd use it to find that one service in a 12-hop request path is the latency culprit. What's the sampling trade-off?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23301_Explain_how_distributed_tracing_trace_IDs_spans_context_prop.md>)
- [Deep] [You detect concept drift on a model served across many regions and want to trigger a retrain. Explain how you'd build a distributed drift-monitoring pipeline that aggregates prediction and feature distributions from every region without a central bottleneck.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23642_You_detect_concept_drift_on_a_model_served_across_many_regio.md>)
- [Deep] [Explain how you'd instrument distributed tracing across a fan-out inference request (gateway to feature store to model to post-processor) to find which hop in a 6-service path is your latency culprit. What's the sampling trade-off at high QPS?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23681_Explain_how_you_d_instrument_distributed_tracing_across_a_fa.md>)
- [Deep] [You need distributed tracing across your own AIOps pipeline (ingest, enrich, detect, correlate, remediate) to find which stage adds latency to time-to-detect. Explain context propagation across async queue hops and the sampling trade-off at high telemetry volume.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23905_You_need_distributed_tracing_across_your_own_AIOps_pipeline_.md>)
- [Deep] [Explain how you'd guarantee that the exact feature values used for a prediction are logged alongside that prediction across a distributed serving fleet, and why reconstructing them after the fact from the offline store is unreliable. What's the throughput and storage design for this at high QPS?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23791_Explain_how_you_d_guarantee_that_the_exact_feature_values_us.md>)
- [Expert] [You're serving an agentic workflow where one user request spawns many chained LLM and tool calls across services. Explain the distributed-tracing, deadline-budgeting, and partial-failure design so a single slow tool call doesn't blow the whole agent's latency or leave orphaned in-flight sub-requests.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23728_You_re_serving_an_agentic_workflow_where_one_user_request_sp.md>)

## Related concepts

- [Fan-out Trade-offs](<../../../System_Design/High_Level_Design/Fan-out_Trade-offs.md>) *(High Level Design)*
- [Provisional and Final Results](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Provisional_and_Final_Results/_index.md>) *(Streaming Architecture)*
- [Sensitivity Points](<../../../Software_Architecture/Trade_off_Analysis/Sensitivity_Points.md>) *(Trade off Analysis)*

## References

- *Cassandra: The Definitive Guide* — Jeff Carpenter, Eben Hewitt — and load testing platform provided by DataStax for Cassandra and DataStax Enterprise. The goal of
- *Ultimate AWS Data Engineering* — Rathish Mohan — Balancing the Load Achieving Performance and Efficiency › Key Metrics for Monitoring Load Balancing and Scaling

