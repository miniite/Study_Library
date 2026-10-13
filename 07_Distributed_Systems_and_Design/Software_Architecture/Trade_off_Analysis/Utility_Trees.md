---
id: "07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Utility_Trees"
level: 5
type: "concept"
topic: "07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis"
tools: []
---

# Utility Trees

**L5 Expert** · [↑ Trade off Analysis](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Practice questions

- [Deep] [An interviewer probes: how is ATAM's utility tree fundamentally different from just writing NFRs into a requirements doc? What does the tree buy you that a flat NFR list structurally cannot?](<../../../PRACTICE/07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Q-03043_An_interviewer_probes_how_is_ATAM_s_utility_tree_fundamental.md>)
- [Deep] [How do you quantify a 'cognitive load' quality attribute for the platform so an ATAM can analyze it — e.g. 'a new SRE can safely execute a cluster upgrade within their first month using only the runbook'? What makes this measurable rather than hand-wavy?](<../../../PRACTICE/07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Q-03230_How_do_you_quantify_a_cognitive_load_quality_attribute_for_t.md>)
- [Deep] [How do you separate quality-attribute requirements the GenAI platform must meet from 'nice-to-have' capabilities teams merely want during utility-tree construction? Why does ATAM force this separation before analysis, and how does it stop the AI roadmap from becoming a demo-driven wishlist?](<../../../PRACTICE/07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Q-03472_How_do_you_separate_quality-attribute_requirements_the_GenAI.md>)
- [Deep] [For a global streaming platform, build the top two levels of a utility tree (quality attributes and their refinements) and justify why you ordered availability and performance the way you did given the business.](<../../../PRACTICE/07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Q-03505_For_a_global_streaming_platform_build_the_top_two_levels_of_.md>)
- [Deep] [How do you separate quality-attribute requirements the AIOps platform must meet from 'nice-to-have' capabilities teams merely want during utility-tree construction? Why does ATAM force this separation before analysis, and how does it stop the observability roadmap from becoming a dashboard-wishlist?](<../../../PRACTICE/07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Q-03579_How_do_you_separate_quality-attribute_requirements_the_AIOps.md>)
- [Expert] [Explain how you would quantify a 'cognitive load' quality attribute for a full-stack service so an ATAM can analyze it — e.g. 'a new engineer ships a safe change to the checkout flow within their first week.' What makes this measurable rather than hand-wavy?](<../../../PRACTICE/07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Q-03073_Explain_how_you_would_quantify_a_cognitive_load_quality_attr.md>)
- [Expert] [You are running an ATAM on a multi-agent orchestration platform where autonomous agents call tools and each other. How do you build the top two levels of the utility tree so that goal-completion reliability, tool-call safety, and bounded cost-per-task each become first-class quality attributes rather than being buried under a vague 'correctness' branch?](<../../../PRACTICE/07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Q-03374_You_are_running_an_ATAM_on_a_multi-agent_orchestration_platf.md>)
- [Expert] [A sensitivity analysis shows a single tuning parameter — the trace tail-sampling rate — dominates both storage cost and the ability to reconstruct a rare incident. How do you document a parameter as an architectural sensitivity point and what follow-up do you demand?](<../../../PRACTICE/07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Q-03548_A_sensitivity_analysis_shows_a_single_tuning_parameter_the_t.md>)

## Related concepts

- [Explainability and Data Access Trade-offs](<../../../11_AI_and_ML/Responsible_AI/AI_Governance/Explainability_and_Data_Access_Trade-offs.md>) *(AI Governance)*
- [Cost and Reliability Correlation](<../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Cost_and_Reliability_Correlation.md>) *(FinOps)*
- [Streaming for AIOps](<../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Provisional_and_Final_Results/Streaming_for_AIOps.md>) *(Streaming Architecture)*
- [Org-wide Lineage and PII Tracking](<../../System_Design/System_Design_Core/Read-through_and_Write-behind_Caching/Org-wide_Lineage_and_PII_Tracking.md>) *(System Design Core)*
- [Data Vault](<../../../02_Data_Engineering/Modeling_and_Warehousing/Modeling_Methodologies/Data_Vault.md>) *(Modeling Methodologies)*
- [Late and Out-of-order Data Strategy](<../../../04_Data_Architecture_and_Strategy/Platform_Strategy/Data_Platform_Strategy/Late_and_Out-of-order_Data_Strategy.md>) *(Data Platform Strategy)*

