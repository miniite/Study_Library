---
id: "05_Data_Governance_and_Security/Governance/Data_Strategy/Centralized_vs_Federated_Telemetry"
level: 5
type: "concept"
topic: "05_Data_Governance_and_Security/Governance/Data_Strategy"
tools: []
---

# Centralized vs Federated Telemetry

**L5 Expert** · [↑ Data Strategy](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Practice questions

- [Intermediate] [How would you set telemetry-governance guardrails for a fast-scaling startup so you don't accumulate observability-data debt you later can't unwind?](<../../../PRACTICE/05_Data_Governance_and_Security/Governance/Data_Strategy/Q-14969_How_would_you_set_telemetry-governance_guardrails_for_a_fast.md>)
- [Deep] [How do you govern the classification of telemetry (public/internal/confidential/security-sensitive) so security logs get stricter access than latency metrics, enforced automatically?](<../../../PRACTICE/05_Data_Governance_and_Security/Governance/Data_Strategy/Q-14942_How_do_you_govern_the_classification_of_telemetry_public_int.md>)
- [Deep] [How do you govern the cost and sprawl of custom metrics so teams can instrument freely but the org isn't paying for millions of never-queried time series?](<../../../PRACTICE/05_Data_Governance_and_Security/Governance/Data_Strategy/Q-14962_How_do_you_govern_the_cost_and_sprawl_of_custom_metrics_so_t.md>)
- [Deep] [What's your strategy for governing which model tier (cheap heuristic vs. expensive ML) is applied to which telemetry-sensitivity class, enforced at pipeline routing time?](<../../../PRACTICE/05_Data_Governance_and_Security/Governance/Data_Strategy/Q-14992_What_s_your_strategy_for_governing_which_model_tier_cheap_he.md>)
- [Deep] [How would you govern the ingestion of business-KPI streams alongside infra telemetry so revenue-impacting incidents are correlated without leaking financials to broad access tiers?](<../../../PRACTICE/05_Data_Governance_and_Security/Governance/Data_Strategy/Q-15013_How_would_you_govern_the_ingestion_of_business-KPI_streams_a.md>)
- [Expert] [How would you decide build-vs-buy for an org-wide telemetry data lake versus a vendor observability platform, and which governance capabilities are non-negotiable either way?](<../../../PRACTICE/05_Data_Governance_and_Security/Governance/Data_Strategy/Q-14928_How_would_you_decide_build-vs-buy_for_an_org-wide_telemetry_.md>)
- [Expert] [What's your strategy for governing cross-region telemetry so EU service data stays in-region while the AIOps control plane gets a unified global view for incident correlation?](<../../../PRACTICE/05_Data_Governance_and_Security/Governance/Data_Strategy/Q-14941_What_s_your_strategy_for_governing_cross-region_telemetry_so.md>)
- [Expert] [What's your approach to governing multi-tenant telemetry in a B2B platform so one customer's operational data can never surface in an AIOps insight served to another?](<../../../PRACTICE/05_Data_Governance_and_Security/Governance/Data_Strategy/Q-14982_What_s_your_approach_to_governing_multi-tenant_telemetry_in_.md>)

## Related concepts

- [Reproducible Telemetry State](<../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Reproducible_Telemetry_State.md>) *(Data Architecture)*
- [Mesh vs Central Observability Lake](<../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Observability_Data_Lake/Mesh_vs_Central_Observability_Lake.md>) *(Observability Data Lake)*
- [Explainability and Data Access Trade-offs](<../../../11_AI_and_ML/Responsible_AI/AI_Governance/Explainability_and_Data_Access_Trade-offs.md>) *(AI Governance)*

