---
id: "07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Regulatory_Requirements_in_Utility_Trees"
level: 5
type: "concept"
topic: "07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis"
tools: []
---

# Regulatory Requirements in Utility Trees

**L5 Expert** · [↑ Trade off Analysis](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Practice questions

- [Deep] [Explain how you would treat non-determinism (variable outputs at temperature > 0) as a first-class quality attribute in the utility tree, and give a scenario whose failure reveals the architecture has no design for bounding output variance across identical requests.](<../../../PRACTICE/07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Q-03382_Explain_how_you_would_treat_non-determinism_variable_outputs.md>)
- [Deep] [A stakeholder proposes a scenario with a response measure of 'the model never hallucinates.' How do you convert that unrealistic absolute into a measurable groundedness/citation-coverage/error-budget target the GenAI architecture can actually be evaluated against?](<../../../PRACTICE/07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Q-03396_A_stakeholder_proposes_a_scenario_with_a_response_measure_of.md>)
- [Deep] [How do you incorporate 'blast-radius containment' into the utility tree such that a failing scenario reveals a shared platform component (the router, the guardrail service, or the prompt registry) can degrade every GenAI product at once? Give the concrete response measure you attach.](<../../../PRACTICE/07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Q-03428_How_do_you_incorporate_blast-radius_containment_into_the_uti.md>)
- [Deep] [How do you handle the situation where the GenAI architecture presented in Phase 1 is the target-state diagram (a clean multi-agent design) while the reality is a pile of prompt-glue in a monolith? Do you evaluate the intended or the as-built system, and how does that shape your findings?](<../../../PRACTICE/07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Q-03453_How_do_you_handle_the_situation_where_the_GenAI_architecture.md>)
- [Deep] [How do you use ATAM findings to negotiate the boundary of an internal SLA between the GenAI platform team and a product team consuming the assistant API? Show how a sensitivity point on model-provider latency becomes a formal dependency contract with a measured response goal.](<../../../PRACTICE/07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Q-03461_How_do_you_use_ATAM_findings_to_negotiate_the_boundary_of_an.md>)
- [Deep] [You are evaluating a GenAI platform where the team has already spent nine months building a bespoke in-house RAG-and-agent framework. How do you run an ATAM that surfaces the real build-vs-buy risks without triggering sunk-cost defensiveness that turns the review into theatre?](<../../../PRACTICE/07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Q-03480_You_are_evaluating_a_GenAI_platform_where_the_team_has_alrea.md>)
- [Expert] [How do you construct a utility tree for a GenAI platform whose business goals conflict — the safety org demands every generation pass a guardrail-and-red-team gate while product demands sub-second first-token latency? Walk me through surfacing that as an explicit tradeoff rather than an organizational grudge.](<../../../PRACTICE/07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Q-03379_How_do_you_construct_a_utility_tree_for_a_GenAI_platform_who.md>)
- [Expert] [A principal peer argues ATAM is obsolete for GenAI because 'we govern with eval suites, guardrail configs, and CI checks now.' Defend or refute: where does a formal ATAM still add value that automated GenAI-governance checks structurally cannot?](<../../../PRACTICE/07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Q-03429_A_principal_peer_argues_ATAM_is_obsolete_for_GenAI_because_w.md>)

## Related concepts

- [Spend Freezes and Prioritization](<../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Spend_Freezes_and_Prioritization.md>) *(FinOps)*
- [Org-wide Lineage and PII Tracking](<../../System_Design/System_Design_Core/Read-through_and_Write-behind_Caching/Org-wide_Lineage_and_PII_Tracking.md>) *(System Design Core)*
- [Synthetic Data Governance](<../../../11_AI_and_ML/Responsible_AI/AI_Governance/Synthetic_Data_Governance.md>) *(AI Governance)*

