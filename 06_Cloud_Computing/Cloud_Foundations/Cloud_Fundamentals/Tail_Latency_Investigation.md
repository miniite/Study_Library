---
id: "06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Tail_Latency_Investigation"
level: 4
type: "concept"
topic: "06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals"
tools: ["AWS", "Prometheus"]
---

# Tail Latency Investigation

**L4 Advanced** · [↑ Cloud Fundamentals](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [AWS](<../../../TOOLS/AWS/_index.md>)
- [Prometheus](<../../../TOOLS/Prometheus/_index.md>)

## Practice questions

- [Intermediate] [A user in Australia complains your web app is slow while US users are fine. How does the cloud let you diagnose and fix it?](<../../../PRACTICE/06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Q-08317_A_user_in_Australia_complains_your_web_app_is_slow_while_US_.md>)
- [Intermediate] [How would you monitor the latency of your LLM API calls in production and alert when responses get too slow?](<../../../PRACTICE/06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Q-08408_How_would_you_monitor_the_latency_of_your_LLM_API_calls_in_p.md>)
- [Intermediate] [Your frontend calls three cloud APIs to render a dashboard and one is slow. How would you keep the page usable and instrument which upstream is the bottleneck from the client side?](<../../../PRACTICE/06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Q-08570_Your_frontend_calls_three_cloud_APIs_to_render_a_dashboard_a.md>)
- [Deep] [Why is a p99 latency number more meaningful than an average when judging your AI service's responsiveness?](<../../../PRACTICE/06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Q-08447_Why_is_a_p99_latency_number_more_meaningful_than_an_average_.md>)
- [Deep] [Your model's accuracy silently dropped in production while latency looked fine. How would you use cloud monitoring to catch quality regressions, not just system health?](<../../../PRACTICE/06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Q-08224_Your_model_s_accuracy_silently_dropped_in_production_while_l.md>)
- [Deep] [Explain how you would trace and fix a memory-per-connection leak that only shows up after a service has been running for a few days in production.](<../../../PRACTICE/06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Q-08616_Explain_how_you_would_trace_and_fix_a_memory-per-connection_.md>)
- [Deep] [Explain how you'd instrument distributed tracing across an API gateway, three services, and a queue so a single request's full latency breakdown is visible.](<../../../PRACTICE/06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Q-08712_Explain_how_you_d_instrument_distributed_tracing_across_an_A.md>)
- [Deep] [Your gp3-backed database hits an IOPS ceiling during nightly reports while CPU is idle. How would you tell it is IOPS and decide between raising provisioned IOPS or moving the report load?](<../../../PRACTICE/06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Q-09185_Your_gp3-backed_database_hits_an_IOPS_ceiling_during_nightly.md>)

## Related concepts

- [Tail Latency After Expansion](<../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Tail_Latency_After_Expansion.md>) *(System Design Core)*
- [Lag-based Consumer Autoscaling](<../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Lag-based_Consumer_Autoscaling.md>) *(Event Driven Architecture)*
- [Aggregation Hiding Tail Problems](<../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Partition_Lag_and_Hot_Keys/Aggregation_Hiding_Tail_Problems.md>) *(Distributed Systems Core)*

