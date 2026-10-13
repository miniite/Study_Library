---
id: "07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Org-wide_Backpressure"
level: 5
type: "deep-dive"
topic: "07_Distributed_Systems_and_Design/System_Design/System_Design_Core"
tools: []
---

# Org-wide Backpressure

**L5 Expert** · [↑ System Design Core](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Practice questions

- [Intermediate] [What is a database connection pool and why is it important?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Q-42264_What_is_a_database_connection_pool_and_why_is_it_important.md>)
- [Deep] [Would you push updates to a live dashboard or let it poll, and what are the trade-offs?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Q-42326_Would_you_push_updates_to_a_live_dashboard_or_let_it_poll_an.md>)
- [Deep] [Your gRPC service's tail latency spikes when a few clients open long-lived streams. Explain HTTP/2 head-of-line blocking, stream multiplexing limits, and how connection pooling changes the picture.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Q-42163_Your_gRPC_service_s_tail_latency_spikes_when_a_few_clients_o.md>)
- [Deep] [Design a rate-limited, cost-aware fan-out to send 10M push notifications in 15 minutes. How do you throttle to respect provider quotas and retry the failures without duplicate sends?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Q-42428_Design_a_rate-limited_cost-aware_fan-out_to_send_10M_push_no.md>)
- [Deep] [A downstream is slow-but-not-down — the worst failure. Explain how bulkheads, per-dependency thread pools, and load-shedding stop it from exhausting your own threads and cascading.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Q-42512_A_downstream_is_slow-but-not-down_the_worst_failure_Explain_.md>)
- [Deep] [Explain why aggressive autoscaling can worsen an incident (scale-out amplifies DB load, cold-start latency). How do you design scaling signals and warm pools so you scale on the right metric?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Q-42544_Explain_why_aggressive_autoscaling_can_worsen_an_incident_sc.md>)
- [Deep] [Do back-of-the-envelope capacity planning for a service at 50k RPS: estimate DB QPS after caching, connection count, cache memory, and network egress. Where do you add safety margin and why?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Q-42574_Do_back-of-the-envelope_capacity_planning_for_a_service_at_5.md>)
- [Deep] [How do you design a system to safely run a load test against production (or a prod-like environment) at 5x current peak without risking real customers? What isolation, data, and safety mechanisms make this responsible?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Q-42652_How_do_you_design_a_system_to_safely_run_a_load_test_against.md>)

## Related concepts

- [Serverless Limits and Throttling](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Serverless_Limits_and_Throttling.md>) *(Cloud Fundamentals)*
- [Fan-out Trade-offs](<../../High_Level_Design/Fan-out_Trade-offs.md>) *(High Level Design)*
- [Slow Dependencies and Load Shedding](<../../../Distributed_Systems/Distributed_Systems_Core/Deadline_Propagation/Slow_Dependencies_and_Load_Shedding.md>) *(Distributed Systems Core)*

