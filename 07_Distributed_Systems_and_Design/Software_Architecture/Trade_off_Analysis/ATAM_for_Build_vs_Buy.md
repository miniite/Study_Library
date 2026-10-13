---
id: "07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/ATAM_for_Build_vs_Buy"
level: 5
type: "concept"
topic: "07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis"
tools: []
---

# ATAM for Build vs Buy

**L5 Expert** · [↑ Trade off Analysis](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Practice questions

- [Deep] [You are asked to run an ATAM across a system-of-systems where the frontend BFF layer, three backend domains, a data lake, and shared platform infra are each owned by a different VP. How do you draw the evaluation boundary so the review is tractable but still catches the cross-cutting integration risks?](<../../../PRACTICE/07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Q-03041_You_are_asked_to_run_an_ATAM_across_a_system-of-systems_wher.md>)
- [Deep] [How do you decide the right business-owner to sit in Phase 1 when the system serves both an internal ops org and external customers with conflicting goals? How does ATAM handle a system with two legitimate but opposed 'business owners'?](<../../../PRACTICE/07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Q-03128_How_do_you_decide_the_right_business-owner_to_sit_in_Phase_1.md>)
- [Deep] [How do you handle the case where two candidate ML serving architectures both satisfy every prioritized scenario and the ATAM does not discriminate? What secondary criteria (framework exit cost, team skill fit, ecosystem maturity) do you bring in, and does that leave ATAM's scope?](<../../../PRACTICE/07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Q-03347_How_do_you_handle_the_case_where_two_candidate_ML_serving_ar.md>)
- [Deep] [How do you run an AIOps platform ATAM under acquisition-integration pressure where two companies' observability stacks (different agents, TSDBs, and incident tools) must interoperate in six months? Which scenarios expose whether integrate-at-the-edge (shared alert bus) or deep-merge is the lower-risk path?](<../../../PRACTICE/07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Q-03539_How_do_you_run_an_AIOps_platform_ATAM_under_acquisition-inte.md>)
- [Deep] [How do you run a database ATAM when the estate spans three vendors' managed services (managed OLTP, managed warehouse, managed cache) and you cannot see inside their internals? Which scenarios still make sense and where does the method break down?](<../../../PRACTICE/07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Q-03647_How_do_you_run_a_database_ATAM_when_the_estate_spans_three_v.md>)
- [Expert] [How do you use ATAM outputs to negotiate the boundary of an SLA with a downstream internal platform team? Show how a sensitivity point on their latency becomes a formal dependency contract with a measured response goal.](<../../../PRACTICE/07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Q-03097_How_do_you_use_ATAM_outputs_to_negotiate_the_boundary_of_an_.md>)
- [Expert] [How do you run an ATAM when the architecture spans systems owned by three different vendors and you cannot see inside their internals? What scenarios still make sense and where does the method break down?](<../../../PRACTICE/07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Q-03172_How_do_you_run_an_ATAM_when_the_architecture_spans_systems_o.md>)
- [Expert] [A principal peer argues ATAM is obsolete for ML work because 'we govern with model cards, eval gates, and CI checks now.' Defend or refute: where does a formal ATAM still add value that automated model-governance checks structurally cannot?](<../../../PRACTICE/07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Q-03317_A_principal_peer_argues_ATAM_is_obsolete_for_ML_work_because.md>)

## Related concepts

- [Org-wide Lineage and PII Tracking](<../../System_Design/System_Design_Core/Read-through_and_Write-behind_Caching/Org-wide_Lineage_and_PII_Tracking.md>) *(System Design Core)*
- [Service Mesh Adoption](<../../System_Design/High_Level_Design/Service_Mesh_Adoption.md>) *(High Level Design)*
- [Design Pattern Governance](<../../System_Design/Low_Level_Design/Over-engineering_vs_Simplicity/Design_Pattern_Governance.md>) *(Low Level Design)*
- [Choosing an Event Backbone](<../../Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Choosing_an_Event_Backbone.md>) *(Distributed Systems Core)*

## References

- *Fundamentals of Software Architecture (2nd ed.)* — Mark Richards, Neal Ford — Analyzing Trade-Offs

