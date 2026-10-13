---
id: "07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Turning_ATAM_Findings_into_Principles"
level: 5
type: "concept"
topic: "07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis"
tools: []
---

# Turning ATAM Findings into Principles

**L5 Expert** · [↑ Trade off Analysis](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Practice questions

- [Deep] [How do you decide whether a discovered risk warrants pausing a launch versus being logged as a tracked debt item? Walk me through the risk-severity rubric you apply during the risk-theme consolidation step of ATAM.](<../../../PRACTICE/07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Q-03055_How_do_you_decide_whether_a_discovered_risk_warrants_pausing.md>)
- [Deep] [How do you fold error budgets and SLO policy into the ATAM utility tree so that an availability scenario is analyzed against the negotiated budget rather than an aspirational 'always up'? Give the concrete response measure you attach.](<../../../PRACTICE/07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Q-03156_How_do_you_fold_error_budgets_and_SLO_policy_into_the_ATAM_u.md>)
- [Deep] [How does Conway's Law show up in an ML platform ATAM? Give an example where a scenario revealed that a feature-pipeline boundary (and its train-serve seam) existed only because of the org chart split between the data-eng and ML teams, and how you documented that as a risk theme.](<../../../PRACTICE/07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Q-03302_How_does_Conway_s_Law_show_up_in_an_ML_platform_ATAM_Give_an.md>)
- [Deep] [How do you handle the political reality that an ATAM's findings threaten a director's flagship in-house LLM fine-tuning program that is already funded and half-built? Walk through protecting the review's integrity and the psychological safety of the engineers who know the quality targets are unreachable.](<../../../PRACTICE/07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Q-03408_How_do_you_handle_the_political_reality_that_an_ATAM_s_findi.md>)
- [Deep] [How do you present ATAM findings to a joint audience of the CTO (cost and reliability-risk framing) and the AIOps platform staff engineers (technical depth) in one session, when the review touched telemetry ingestion, detection models, and auto-remediation? What layered artifact do you produce?](<../../../PRACTICE/07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Q-03524_How_do_you_present_ATAM_findings_to_a_joint_audience_of_the_.md>)
- [Expert] [Describe how you would fold ATAM into a continuous architecture governance model — running mini-evaluations at RFC time — so major risks are caught before the full formal review. What triggers a full ATAM versus a lightweight scenario check?](<../../../PRACTICE/07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Q-03040_Describe_how_you_would_fold_ATAM_into_a_continuous_architect.md>)
- [Expert] [Explain how you would treat train-serve skew as a first-class quality attribute in the utility tree, and give a scenario whose failure reveals the feature-computation architecture silently diverges between offline training and online serving paths.](<../../../PRACTICE/07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Q-03271_Explain_how_you_would_treat_train-serve_skew_as_a_first-clas.md>)
- [Expert] [How do you fold model-provider dependency risk into the utility tree so a scenario reveals what happens when your sole frontier-model vendor deprecates a model, raises price, or degrades quality? What response measure do you attach to 'time-to-swap-provider'?](<../../../PRACTICE/07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Q-03395_How_do_you_fold_model-provider_dependency_risk_into_the_util.md>)

## Related concepts

- [Explainability and Data Access Trade-offs](<../../../11_AI_and_ML/Responsible_AI/AI_Governance/Explainability_and_Data_Access_Trade-offs.md>) *(AI Governance)*
- [Model Routing Cost vs Quality](<../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Model_Routing_Cost_vs_Quality.md>) *(FinOps)*
- [Mixed Version Fleets](<../../Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/_index.md>) *(Distributed Systems Core)*
- [Org-wide Lineage and PII Tracking](<../../System_Design/System_Design_Core/Read-through_and_Write-behind_Caching/Org-wide_Lineage_and_PII_Tracking.md>) *(System Design Core)*
- [Design Pattern Governance](<../../System_Design/Low_Level_Design/Over-engineering_vs_Simplicity/Design_Pattern_Governance.md>) *(Low Level Design)*
- [Deployment Gating Trade-offs](<../../../10_DevOps_and_Platform/Delivery/CI_CD/Shared_Pipeline_Changes/Deployment_Gating_Trade-offs.md>) *(CI CD)*

