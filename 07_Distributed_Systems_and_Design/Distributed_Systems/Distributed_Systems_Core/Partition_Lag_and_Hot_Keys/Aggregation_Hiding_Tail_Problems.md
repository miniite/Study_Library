---
id: "07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Aggregation_Hiding_Tail_Problems"
level: 5
type: "deep-dive"
topic: "07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core"
tools: []
---

# Aggregation Hiding Tail Problems

**L5 Expert** · [↑ Distributed Systems Core](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Practice questions

- [Intermediate] [Your LLM cost dashboard shows average latency is fine but a specific customer complains of slowness. Explain why aggregate latency hides per-tenant tail behavior in token generation and how you'd instrument monitoring to surface it.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23176_Your_LLM_cost_dashboard_shows_average_latency_is_fine_but_a_.md>)
- [Intermediate] [Your on-call got paged because averaged latency dashboards looked fine but customers reported slowness. Explain why aggregate metrics hide per-tenant tail latency and how you'd instrument to surface it.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23463_Your_on-call_got_paged_because_averaged_latency_dashboards_l.md>)
- [Deep] [Your monitoring dashboards show model accuracy is fine on average but a specific customer segment is silently degraded. Explain why aggregate metrics hide per-segment tail behavior and how you'd instrument distributed monitoring to surface it.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23644_Your_monitoring_dashboards_show_model_accuracy_is_fine_on_av.md>)
- [Deep] [You detect that anomaly recall silently dropped for one customer segment while aggregate detection metrics look healthy. Explain why averaged metrics hide per-segment tail behavior and how you'd instrument distributed monitoring to surface the per-segment degradation.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23897_You_detect_that_anomaly_recall_silently_dropped_for_one_cust.md>)
- [Deep] [Your incident-automation fans a diagnostic query out to 50 hosts and merges results, and p99.9 time-to-diagnose is dominated by the slowest host. Explain tail-latency amplification in fan-out and how hedged or tied requests help, plus the extra-load cap you'd set.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23930_Your_incident-automation_fans_a_diagnostic_query_out_to_50_h.md>)
- [Deep] [Your averaged pipeline-latency dashboard looks fine but one downstream team reports stale data for their partition. Explain why aggregate metrics hide per-partition tail behavior and how you'd instrument per-partition or per-tenant freshness.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-24339_Your_averaged_pipeline-latency_dashboard_looks_fine_but_one_.md>)
- [Deep] [Explain coordinated omission and why the org's load-test dashboards keep lying about tail latency while production burns. Redesign both the load generator and the production measurement so p99.9 reflects what users actually experience.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23564_Explain_coordinated_omission_and_why_the_org_s_load-test_das.md>)
- [Deep] [A principal-loop probe: your dashboards show healthy averaged latency but a specific customer segment is silently degraded and the AIOps system never flagged it. Explain why aggregate metrics and coordinated omission hide per-segment tail behavior and how you'd instrument distributed monitoring to surface it.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23967_A_principal-loop_probe_your_dashboards_show_healthy_averaged.md>)

## Related concepts

- [Tail Latency Investigation](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Tail_Latency_Investigation.md>) *(Cloud Fundamentals)*
- [Tail Latency After Expansion](<../../../System_Design/System_Design_Core/Tail_Latency_After_Expansion.md>) *(System Design Core)*
- [Sensitivity Points](<../../../Software_Architecture/Trade_off_Analysis/Sensitivity_Points.md>) *(Trade off Analysis)*

