---
id: "10_DevOps_and_Platform/Delivery/CI_CD/Data_and_Analytics_CI_Gates"
level: 4
type: "concept"
topic: "10_DevOps_and_Platform/Delivery/CI_CD"
tools: ["dbt", "Git_and_GitHub", "Python"]
---

# Data and Analytics CI Gates

**L4 Advanced** · [↑ CI CD](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [dbt](<../../../TOOLS/dbt/_index.md>)
- [Git and GitHub](<../../../TOOLS/Git_and_GitHub/_index.md>)
- [Python](<../../../TOOLS/Python/_index.md>)

## Practice questions

- [Basic] [Why might a team want to validate that a Grafana dashboard JSON file is well-formed as a CI step before deploying it?](<../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06431_Why_might_a_team_want_to_validate_that_a_Grafana_dashboard_J.md>)
- [Intermediate] [How would you build a CI check that validates a model card / metadata (intended use, metrics, data lineage) is present and complete before promotion?](<../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07641_How_would_you_build_a_CI_check_that_validates_a_model_card_m.md>)
- [Deep] [How would you build a CI check that a new prompt keeps output in the required JSON schema the downstream service expects, given the model can drift into malformed output?](<../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06381_How_would_you_build_a_CI_check_that_a_new_prompt_keeps_outpu.md>)
- [Deep] [How would you deploy a change to your log-to-metric extraction so a bad regex can't spike cardinality and blow up your time-series database in prod?](<../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06585_How_would_you_deploy_a_change_to_your_log-to-metric_extracti.md>)
- [Deep] [Your dbt slim-CI build relies on a production manifest.json artifact and it went stale after a hotfix ran outside CI; how would you keep the deferred state manifest trustworthy so the diff isn't wrong?](<../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06652_Your_dbt_slim-CI_build_relies_on_a_production_manifest_json_.md>)
- [Deep] [How would you gate a dbt PR on a cost/scan-bytes check so a model that would scan terabytes in BigQuery can't merge without review?](<../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06675_How_would_you_gate_a_dbt_PR_on_a_cost_scan-bytes_check_so_a_.md>)
- [Deep] [How would you make CI enforce that every new dbt/Spark table has an owner, a description, freshness monitoring, and a data-classification tag before it can be deployed?](<../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06737_How_would_you_make_CI_enforce_that_every_new_dbt_Spark_table.md>)
- [Deep] [A downstream service consumes your model's score field and you changed its range from 0-1 to logits; how would CI have caught this contract break?](<../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07701_A_downstream_service_consumes_your_model_s_score_field_and_y.md>)

## Related concepts

- [dbt Models and Runs](<../../../TOOLS/dbt/dbt_Core/dbt_Models_and_Runs.md>) *(dbt Core)*
- [Automated Dependency Updates](<../../../TOOLS/Git_and_GitHub/Git_Core/Automated_Dependency_Updates.md>) *(Git Core)*
- [Isolating CI Environments](<../../../TOOLS/Git_and_GitHub/Git_Core/Isolating_CI_Environments.md>) *(Git Core)*
- [Securing ML Delivery](<../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Safe_Config_Rollouts/Securing_ML_Delivery.md>) *(Cloud Fundamentals)*
- [Distinguishing Drift from Pipeline Breaks](<../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Distinguishing_Drift_from_Pipeline_Breaks.md>) *(Data Architecture)*

