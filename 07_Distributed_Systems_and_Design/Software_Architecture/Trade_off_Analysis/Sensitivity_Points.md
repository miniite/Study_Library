---
id: "07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Sensitivity_Points"
level: 5
type: "concept"
topic: "07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis"
tools: []
---

# Sensitivity Points

**L5 Expert** · [↑ Trade off Analysis](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Practice questions

- [Intermediate] [A stakeholder proposes a scenario with a response measure of 'zero failed deploys, ever.' How do you convert that unrealistic absolute into a measurable change-failure-rate and rollback-time target the pipeline architecture can actually be evaluated against?](<../../../PRACTICE/07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Q-03175_A_stakeholder_proposes_a_scenario_with_a_response_measure_of.md>)
- [Deep] [A scenario tests behavior under a retry-storm that cascades from a failing downstream into connection-pool and node-resource exhaustion fleet-wide. How do you construct the scenario so it forces the retry-budget and circuit-breaker sensitivity points into the open?](<../../../PRACTICE/07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Q-03178_A_scenario_tests_behavior_under_a_retry-storm_that_cascades_.md>)
- [Deep] [A scenario tests graceful degradation when the platform's central logging/metrics pipeline is saturated during an incident. Walk through analyzing whether the platform degrades gracefully (drops sampling) or fails closed (blocks workloads), and what sensitivity points that exposes.](<../../../PRACTICE/07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Q-03225_A_scenario_tests_graceful_degradation_when_the_platform_s_ce.md>)
- [Deep] [A scenario tests graceful degradation when the real-time feature-enrichment pipeline is saturated during a traffic spike. Walk through analyzing whether the serving path degrades gracefully (falls back to cached/default features) or fails closed (blocks scoring), and what sensitivity points that exposes.](<../../../PRACTICE/07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Q-03343_A_scenario_tests_graceful_degradation_when_the_real-time_fea.md>)
- [Deep] [A tradeoff point pits a real-time streaming detection path (fast MTTD, high infra cost) against a micro-batch path (cheap, minutes of detection lag). How do you quantify both response measures so a VP of SRE decides on evidence rather than the loudest advocate?](<../../../PRACTICE/07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Q-03487_A_tradeoff_point_pits_a_real-time_streaming_detection_path_f.md>)
- [Deep] [A sensitivity analysis shows that a single configuration parameter (thread-pool size) is the dominant knob for both throughput and stability under load. How do you document a parameter as an architectural sensitivity point and what follow-up do you demand?](<../../../PRACTICE/07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Q-03666_A_sensitivity_analysis_shows_that_a_single_configuration_par.md>)
- [Expert] [Give a concrete example of a scenario where satisfying a disaster-recovery RPO goal directly trades off against write latency, and walk through how you would present that tradeoff point with numbers rather than adjectives.](<../../../PRACTICE/07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Q-03216_Give_a_concrete_example_of_a_scenario_where_satisfying_a_dis.md>)
- [Expert] [A scenario reveals that scaling to 100x current load would require re-sharding a stateful service with no online resharding path today. How do you record this — risk, tradeoff, or sensitivity — and how do you frame the remediation roadmap?](<../../../PRACTICE/07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Q-03427_A_scenario_reveals_that_scaling_to_100x_current_load_would_r.md>)

## Related concepts

- [Org-wide Lineage and PII Tracking](<../../System_Design/System_Design_Core/Read-through_and_Write-behind_Caching/Org-wide_Lineage_and_PII_Tracking.md>) *(System Design Core)*
- [Design Pattern Governance](<../../System_Design/Low_Level_Design/Over-engineering_vs_Simplicity/Design_Pattern_Governance.md>) *(Low Level Design)*
- [Backpressure and Load Shedding](<../../System_Design/High_Level_Design/Backpressure_and_Load_Shedding.md>) *(High Level Design)*
- [Fan-out Trade-offs](<../../System_Design/High_Level_Design/Fan-out_Trade-offs.md>) *(High Level Design)*
- [Graceful Degradation in Streaming](<../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Provisional_and_Final_Results/Graceful_Degradation_in_Streaming.md>) *(Streaming Architecture)*

