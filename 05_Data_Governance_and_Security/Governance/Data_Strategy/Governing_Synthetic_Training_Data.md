---
id: "05_Data_Governance_and_Security/Governance/Data_Strategy/Governing_Synthetic_Training_Data"
level: 5
type: "concept"
topic: "05_Data_Governance_and_Security/Governance/Data_Strategy"
tools: ["AWS", "Apache_Kafka"]
---

# Governing Synthetic Training Data

**L5 Expert** · [↑ Data Strategy](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [AWS](<../../../TOOLS/AWS/_index.md>)
- [Apache Kafka](<../../../TOOLS/Apache_Kafka/_index.md>)

## Practice questions

- [Deep] [What is your strategy for lineage tracking across a feature store when features are derived from other features, and how do you make that lineage actionable during an incident?](<../../../PRACTICE/05_Data_Governance_and_Security/Governance/Data_Strategy/Q-14745_What_is_your_strategy_for_lineage_tracking_across_a_feature_.md>)
- [Deep] [Explain how you'd unify metadata across a fragmented stack (Kafka, Spark, Snowflake, S3, feature store) into a single lineage graph teams actually trust.](<../../../PRACTICE/05_Data_Governance_and_Security/Governance/Data_Strategy/Q-14772_Explain_how_you_d_unify_metadata_across_a_fragmented_stack_K.md>)
- [Deep] [How do you approach governance for real-time streaming features where you can't easily snapshot data, yet you still need reproducibility and auditability?](<../../../PRACTICE/05_Data_Governance_and_Security/Governance/Data_Strategy/Q-14780_How_do_you_approach_governance_for_real-time_streaming_featu.md>)
- [Deep] [What's your approach to governing the reuse of model-generated content as future training data, to avoid model collapse and untraceable synthetic-lineage sprawl?](<../../../PRACTICE/05_Data_Governance_and_Security/Governance/Data_Strategy/Q-14846_What_s_your_approach_to_governing_the_reuse_of_model-generat.md>)
- [Deep] [What's your strategy for governing the retirement of an AIOps model such that its training telemetry, features, and lineage are archived or purged per policy without breaking dependents?](<../../../PRACTICE/05_Data_Governance_and_Security/Governance/Data_Strategy/Q-14998_What_s_your_strategy_for_governing_the_retirement_of_an_AIOp.md>)
- [Expert] [When a regulator asks you to prove exactly which data trained a specific model version deployed 8 months ago, what governance infrastructure must already be in place?](<../../../PRACTICE/05_Data_Governance_and_Security/Governance/Data_Strategy/Q-14750_When_a_regulator_asks_you_to_prove_exactly_which_data_traine.md>)
- [Expert] [When you inherit an org with 15 shadow data pipelines feeding ML models nobody governs, what's your first 90-day plan to bring them under governance without breaking prod?](<../../../PRACTICE/05_Data_Governance_and_Security/Governance/Data_Strategy/Q-14768_When_you_inherit_an_org_with_15_shadow_data_pipelines_feedin.md>)
- [Expert] [What's your framework for governing data-lineage across a hybrid pipeline mixing Spark batch, Kafka streams, and vector-index refreshes into one trustworthy graph?](<../../../PRACTICE/05_Data_Governance_and_Security/Governance/Data_Strategy/Q-14882_What_s_your_framework_for_governing_data-lineage_across_a_hy.md>)

## Related concepts

- [Governing Real-time Features](<../Stewardship/Governing_Real-time_Features.md>) *(Stewardship)*
- [Distinguishing Drift from Pipeline Breaks](<../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Distinguishing_Drift_from_Pipeline_Breaks.md>) *(Data Architecture)*
- [Lineage for Dynamic SQL](<../Lineage/Lineage_for_Dynamic_SQL.md>) *(Lineage)*

