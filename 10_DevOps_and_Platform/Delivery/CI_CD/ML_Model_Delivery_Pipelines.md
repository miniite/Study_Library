---
id: "10_DevOps_and_Platform/Delivery/CI_CD/ML_Model_Delivery_Pipelines"
level: 4
type: "concept"
topic: "10_DevOps_and_Platform/Delivery/CI_CD"
tools: ["Git_and_GitHub", "Docker", "Python", "Kubernetes"]
---

# ML Model Delivery Pipelines

**L4 Advanced** · [↑ CI CD](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Git and GitHub](<../../../TOOLS/Git_and_GitHub/_index.md>)
- [Docker](<../../../TOOLS/Docker/_index.md>)
- [Python](<../../../TOOLS/Python/_index.md>)
- [Kubernetes](<../../../TOOLS/Kubernetes/_index.md>)

## Practice questions

- [Intermediate] [How would you set a CI budget and concurrency policy for nightly model retrains so a burst of retrains across many services doesn't starve regular deploy pipelines?](<../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06641_How_would_you_set_a_CI_budget_and_concurrency_policy_for_nig.md>)
- [Deep] [Your guardrail/safety classifier is deployed independently of the main LLM; how do you keep their versions compatible through CI so a guardrail update never silently starts blocking valid outputs?](<../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06332_Your_guardrail_safety_classifier_is_deployed_independently_o.md>)
- [Deep] [Your batch inference pipeline (nightly summaries) is a deploy target too; how would you version and roll back a batch LLM job whose outputs have already been written downstream?](<../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06418_Your_batch_inference_pipeline_nightly_summaries_is_a_deploy_.md>)
- [Deep] [Your detector's config and its model are deployed by separate pipelines and a race sometimes pairs a new model with an old threshold config; how would you enforce ordering across the two?](<../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06633_Your_detector_s_config_and_its_model_are_deployed_by_separat.md>)
- [Deep] [How would you trigger an automated retraining pipeline when monitoring detects data drift, and what guardrails stop it from shipping a worse model?](<../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07629_How_would_you_trigger_an_automated_retraining_pipeline_when_.md>)
- [Deep] [Why did you choose an ML-specific pipeline tool (Kubeflow/Vertex/SageMaker Pipelines) over plain GitHub Actions for a model workflow, in your experience?](<../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07689_Why_did_you_choose_an_ML-specific_pipeline_tool_Kubeflow_Ver.md>)
- [Expert] [How would you canary a new version of a batch pipeline by running it in shadow (writing to a side table) and comparing outputs before promoting it to write the real table?](<../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06683_How_would_you_canary_a_new_version_of_a_batch_pipeline_by_ru.md>)
- [Expert] [You changed the embedding model; walk me through the pipeline that re-embeds the corpus, swaps the index, and cuts over search without a mixed-embedding window.](<../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07647_You_changed_the_embedding_model_walk_me_through_the_pipeline.md>)

## Related concepts

- [Shared Pipeline Platforms](<../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Orchestration_vs_Processing/Shared_Pipeline_Platforms.md>) *(Pipeline Orchestration)*
- [Securing ML Delivery](<../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Safe_Config_Rollouts/Securing_ML_Delivery.md>) *(Cloud Fundamentals)*
- [Multi-cloud Strategy](<../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Managed_Services_vs_Self-hosting/Multi-cloud_Strategy.md>) *(Cloud Fundamentals)*

