---
id: "07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/ATAM_for_Platform_Decisions"
level: 5
type: "concept"
topic: "07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis"
tools: []
---

# ATAM for Platform Decisions

**L5 Expert** · [↑ Trade off Analysis](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Practice questions

- [Intermediate] [A scenario tests whether the platform can enforce that every workload has resource requests/limits and a PodDisruptionBudget. During analysis you find enforcement is advisory, not blocking. How significant is this finding and how do you drive it to closure?](<../../../PRACTICE/07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Q-03243_A_scenario_tests_whether_the_platform_can_enforce_that_every.md>)
- [Deep] [How do you keep a platform ATAM from devolving into a Kubernetes-astronaut debate about the theoretically ideal mesh/CNI/operator combo? What facilitation discipline anchors every scenario to a real, prioritized business driver like time-to-recover or cost-per-team?](<../../../PRACTICE/07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Q-03228_How_do_you_keep_a_platform_ATAM_from_devolving_into_a_Kubern.md>)
- [Deep] [How do you use ATAM to evaluate a proposal to adopt a lakehouse (Delta/Iceberg) as the single ML data-and-feature substrate versus keeping separate warehouse and streaming stores? Which scenarios expose the freshness, point-in-time-correctness, and operational-complexity tradeoffs?](<../../../PRACTICE/07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Q-03358_How_do_you_use_ATAM_to_evaluate_a_proposal_to_adopt_a_lakeho.md>)
- [Deep] [How do you use ATAM to decide whether to adopt a platform-wide alerting policy gate (every new alert rule must pass a signal-to-noise and runbook-linkage check before activation) versus trusting teams to self-certify? Which scenarios expose whether the gate buys real alert-fatigue-vs-velocity gains?](<../../../PRACTICE/07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Q-03522_How_do_you_use_ATAM_to_decide_whether_to_adopt_a_platform-wi.md>)
- [Deep] [How do you use ATAM to adjudicate whether the OLTP core should run on a self-managed PostgreSQL fleet versus a managed distributed-SQL service? Which scenarios best reveal whether the buy option actually satisfies your HA, patch-latency, and exit-cost goals?](<../../../PRACTICE/07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Q-03603_How_do_you_use_ATAM_to_adjudicate_whether_the_OLTP_core_shou.md>)
- [Expert] [Explain how you would run an ATAM whose primary goal is to justify (or kill) a platform-team's request to migrate every team onto Kubernetes. Which scenarios reveal whether the operational complexity buys real quality-attribute gains for product teams?](<../../../PRACTICE/07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Q-03090_Explain_how_you_would_run_an_ATAM_whose_primary_goal_is_to_j.md>)
- [Expert] [A tradeoff sits between enforcing a golden-path paved road that most teams must use (consistency, security baseline) and allowing off-road escapes for edge cases (velocity, innovation). How do you scenario-test the standardization-vs-flexibility tension for an IDP serving 40 teams?](<../../../PRACTICE/07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Q-03234_A_tradeoff_sits_between_enforcing_a_golden-path_paved_road_t.md>)
- [Expert] [How do you use ATAM to evaluate whether to adopt a semantic caching layer (return cached generations for near-duplicate prompts) versus always calling the model fresh? Which scenarios expose the cost-savings-vs-staleness-vs-privacy-leak tradeoffs this creates?](<../../../PRACTICE/07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Q-03456_How_do_you_use_ATAM_to_evaluate_whether_to_adopt_a_semantic_.md>)

## Related concepts

- [Fan-out Trade-offs](<../../System_Design/High_Level_Design/Fan-out_Trade-offs.md>) *(High Level Design)*
- [Model Routing Cost vs Quality](<../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Model_Routing_Cost_vs_Quality.md>) *(FinOps)*
- [GPU Quota Management](<../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/GPU_Sharing/GPU_Quota_Management.md>) *(Cloud Fundamentals)*
- [API Composition and BFF](<../../System_Design/System_Design_Core/Idempotent_Order_APIs/API_Composition_and_BFF.md>) *(System Design Core)*
- [Choosing an Event Backbone](<../../Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Choosing_an_Event_Backbone.md>) *(Distributed Systems Core)*
- [Org-wide Lineage and PII Tracking](<../../System_Design/System_Design_Core/Read-through_and_Write-behind_Caching/Org-wide_Lineage_and_PII_Tracking.md>) *(System Design Core)*

