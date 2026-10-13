---
id: "11_AI_and_ML/Responsible_AI/AI_Governance/Explainability_and_Data_Access_Trade-offs"
level: 5
type: "concept"
topic: "11_AI_and_ML/Responsible_AI/AI_Governance"
tools: []
---

# Explainability and Data Access Trade-offs

**L5 Expert** · [↑ AI Governance](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Practice questions

- [Deep] [How do you quantify and budget the latency/cost overhead of running safety classifiers and guardrails inline, and when do you move them off the critical path?](<../../../PRACTICE/11_AI_and_ML/Responsible_AI/AI_Governance/Q-00914_How_do_you_quantify_and_budget_the_latency_cost_overhead_of_.md>)
- [Deep] [Your incident-automation platform uses ML to auto-remediate production alerts. How do you govern the blast radius so a mis-triaged model can't cascade a bad remediation across the fleet?](<../../../PRACTICE/11_AI_and_ML/Responsible_AI/AI_Governance/Q-01027_Your_incident-automation_platform_uses_ML_to_auto-remediate_.md>)
- [Deep] [How do you govern an alert-correlation model so that its grouping decisions are explainable enough for an incident commander to trust or override during a live sev-1?](<../../../PRACTICE/11_AI_and_ML/Responsible_AI/AI_Governance/Q-01040_How_do_you_govern_an_alert-correlation_model_so_that_its_gro.md>)
- [Deep] [What's your framework for governing which telemetry attributes an AIOps model may use as features, excluding proxies that could correlate model behavior with a specific team, region, or customer unfairly?](<../../../PRACTICE/11_AI_and_ML/Responsible_AI/AI_Governance/Q-01074_What_s_your_framework_for_governing_which_telemetry_attribut.md>)
- [Deep] [How do you govern the balance between an AIOps model's need for low-latency inline decisioning and the desire to run a safety/guardrail check that adds latency to every remediation?](<../../../PRACTICE/11_AI_and_ML/Responsible_AI/AI_Governance/Q-01100_How_do_you_govern_the_balance_between_an_AIOps_model_s_need_.md>)
- [Deep] [How do you govern the data lineage of derived telemetry (rolled-up metrics, sampled traces) so an AIOps model's decisions can be traced back to the raw signals despite aggregation and sampling?](<../../../PRACTICE/11_AI_and_ML/Responsible_AI/AI_Governance/Q-01114_How_do_you_govern_the_data_lineage_of_derived_telemetry_roll.md>)
- [Deep] [When a fairness constraint and an accuracy target conflict, how do you frame the trade-off for a non-technical exec committee and translate their decision back into a training objective?](<../../../PRACTICE/11_AI_and_ML/Responsible_AI/AI_Governance/Q-01124_When_a_fairness_constraint_and_an_accuracy_target_conflict_h.md>)
- [Expert] [How would you govern the tension between low-latency streaming responses and the need to run output-safety checks that ideally see the full completion before releasing it?](<../../../PRACTICE/11_AI_and_ML/Responsible_AI/AI_Governance/Q-00993_How_would_you_govern_the_tension_between_low-latency_streami.md>)

## Related concepts

- [Governing Platform Telemetry](<../../../05_Data_Governance_and_Security/Governance/Data_Strategy/Governing_Platform_Telemetry.md>) *(Data Strategy)*
- [Certifying Data Sources](<../../../05_Data_Governance_and_Security/Governance/Data_Strategy/Certifying_Data_Sources.md>) *(Data Strategy)*
- [Centralized vs Federated Telemetry](<../../../05_Data_Governance_and_Security/Governance/Data_Strategy/Centralized_vs_Federated_Telemetry.md>) *(Data Strategy)*

