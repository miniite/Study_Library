---
id: "10_DevOps_and_Platform/Delivery/CI_CD/Infrastructure_Drift_Detection"
level: 4
type: "concept"
topic: "10_DevOps_and_Platform/Delivery/CI_CD"
tools: []
---

# Infrastructure Drift Detection

**L4 Advanced** · [↑ CI CD](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Practice questions

- [Basic] [What does 'shift left' mean when applied to catching a bad alert rule early in a pipeline rather than in production?](<../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06460_What_does_shift_left_mean_when_applied_to_catching_a_bad_ale.md>)
- [Deep] [How would you catch, before deploy, that a change to your context-truncation logic causes the model to drop the retrieved evidence and hallucinate more?](<../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06423_How_would_you_catch_before_deploy_that_a_change_to_your_cont.md>)
- [Deep] [Your CI runs the anomaly detector against a fixed labeled dataset, but prod telemetry drifted and the dataset is stale; how would you keep the evaluation dataset representative over time?](<../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06564_Your_CI_runs_the_anomaly_detector_against_a_fixed_labeled_da.md>)
- [Deep] [Your ML-on-telemetry job runs as a scheduled pipeline; how would you deploy a change to the feature pipeline without corrupting the feature store the online detector reads from?](<../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06577_Your_ML-on-telemetry_job_runs_as_a_scheduled_pipeline_how_wo.md>)
- [Deep] [A pipeline deployed a new detector that shares the TSDB with others and its heavy queries slowed everyone's alerts; how would you test the query-cost impact of a detector change before deploy?](<../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06613_A_pipeline_deployed_a_new_detector_that_shares_the_TSDB_with.md>)
- [Deep] [How would you catch a deploy that increases your alerting pipeline's end-to-end detection latency (ingest to page) beyond your SLO, before it hits on-call in prod?](<../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06630_How_would_you_catch_a_deploy_that_increases_your_alerting_pi.md>)
- [Deep] [Your team deploys DAGs and dbt models from the same monorepo; how would you build affected-target detection so a change to one model tests only its lineage, not the whole warehouse?](<../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06736_Your_team_deploys_DAGs_and_dbt_models_from_the_same_monorepo.md>)
- [Deep] [How would you detect at the pipeline level that a dependency upgrade changed behavior in a way tests don't cover (e.g., a JSON serialization default flipped), before it causes subtle prod bugs?](<../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07043_How_would_you_detect_at_the_pipeline_level_that_a_dependency.md>)

## Related concepts

- [Adaptive Thresholds](<../../../11_AI_and_ML/Applied_ML/Anomaly_Detection/Adaptive_Thresholds.md>) *(Anomaly Detection)*
- [Shared Pipeline Platforms](<../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Orchestration_vs_Processing/Shared_Pipeline_Platforms.md>) *(Pipeline Orchestration)*
- [Verifying Live Migrations](<../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Verifying_Live_Migrations.md>) *(Distributed Systems Core)*

