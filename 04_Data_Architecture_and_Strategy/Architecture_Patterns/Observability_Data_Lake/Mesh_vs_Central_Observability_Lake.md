---
id: "04_Data_Architecture_and_Strategy/Architecture_Patterns/Observability_Data_Lake/Mesh_vs_Central_Observability_Lake"
level: 5
type: "concept"
topic: "04_Data_Architecture_and_Strategy/Architecture_Patterns/Observability_Data_Lake"
tools: ["AWS"]
---

# Mesh vs Central Observability Lake

**L5 Expert** · [↑ Observability Data Lake](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [AWS](<../../../TOOLS/AWS/_index.md>)

## Practice questions

- [Intermediate] [Describe how you'd unify metric naming and label conventions across teams before their telemetry lands in a shared lake.](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Observability_Data_Lake/Q-34466_Describe_how_you_d_unify_metric_naming_and_label_conventions.md>)
- [Deep] [How would you build change-data-capture from your config and CMDB into the lake so incident timelines include infrastructure changes?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Observability_Data_Lake/Q-34471_How_would_you_build_change-data-capture_from_your_config_and.md>)
- [Deep] [How would you architect a policy engine that enforces sampling and cardinality budgets at the collector edge so cost control happens before data ever hits the lake?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Observability_Data_Lake/Q-34543_How_would_you_architect_a_policy_engine_that_enforces_sampli.md>)
- [Deep] [What is your strategy for integrating CMDB config and deployment change events into the lake so incident timelines and RCA models have change context first-class?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Observability_Data_Lake/Q-34559_What_is_your_strategy_for_integrating_CMDB_config_and_deploy.md>)
- [Expert] [How do you balance building a central observability lake versus per-team lakes and what governance problems does centralization solve or create?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Observability_Data_Lake/Q-34435_How_do_you_balance_building_a_central_observability_lake_ver.md>)
- [Expert] [Walk me through the FinOps model you would build so that observability-lake spend is allocated back to teams in a way that actually changes their instrumentation behavior.](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Observability_Data_Lake/Q-34504_Walk_me_through_the_FinOps_model_you_would_build_so_that_obs.md>)
- [Expert] [How would you phase the introduction of an LLM-based incident copilot on top of the lake so it earns trust before it's allowed to trigger any automated action?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Observability_Data_Lake/Q-34526_How_would_you_phase_the_introduction_of_an_LLM-based_inciden.md>)
- [Expert] [How do you build a governance council that decides which new AIOps use cases are allowed to write derived data back into the shared lake?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Observability_Data_Lake/Q-34541_How_do_you_build_a_governance_council_that_decides_which_new.md>)

## Related concepts

- [Centralized vs Mesh Observability Data](<../Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Centralized_vs_Mesh_Observability_Data.md>) *(Data Architecture)*
- [Centralized vs Federated Telemetry](<../../../05_Data_Governance_and_Security/Governance/Data_Strategy/Centralized_vs_Federated_Telemetry.md>) *(Data Strategy)*
- [Reproducible Telemetry State](<../Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Reproducible_Telemetry_State.md>) *(Data Architecture)*

