---
id: "10_DevOps_and_Platform/Delivery/CI_CD/Test_Data_Governance"
level: 5
type: "deep-dive"
topic: "10_DevOps_and_Platform/Delivery/CI_CD"
tools: ["AWS", "Git_and_GitHub", "Python", "GitLab", "Kubernetes", "Jenkins"]
---

# Test Data Governance

**L5 Expert** · [↑ CI CD](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [AWS](<../../../../TOOLS/AWS/_index.md>)
- [Git and GitHub](<../../../../TOOLS/Git_and_GitHub/_index.md>)
- [Python](<../../../../TOOLS/Python/_index.md>)
- [GitLab](<../../../../TOOLS/GitLab/_index.md>)
- [Kubernetes](<../../../../TOOLS/Kubernetes/_index.md>)
- [Jenkins](<../../../../TOOLS/Jenkins/_index.md>)

## Practice questions

- [Basic] [What is a build artifact in the context of packaging an anomaly-detection model, and where would the pipeline store it?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06494_What_is_a_build_artifact_in_the_context_of_packaging_an_anom.md>)
- [Deep] [An eval dataset lives in the repo and drifts from production traffic; how would you build a pipeline step that periodically refreshes the golden set from real logs without leaking PII into CI?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06334_An_eval_dataset_lives_in_the_repo_and_drifts_from_production.md>)
- [Deep] [How would you make a CI pipeline detect and block a data pipeline change that would write PII to an unencrypted or wrongly-classified table, using policy-as-code?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06720_How_would_you_make_a_CI_pipeline_detect_and_block_a_data_pip.md>)
- [Deep] [How do you architect a pipeline so that the difference between 'the build is reproducible' and 'the deploy is reproducible' is explicit, and what fails silently when teams conflate the two?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07166_How_do_you_architect_a_pipeline_so_that_the_difference_betwe.md>)
- [Expert] [How would you make terraform plan on a PR trustworthy as a review artifact, and what stops a plan-then-apply race where infra changes between the two steps?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06856_How_would_you_make_terraform_plan_on_a_PR_trustworthy_as_a_r.md>)
- [Expert] [Data governance meets CI/CD: pipelines routinely provision test databases seeded from production, creating a compliance exposure. Architect a data strategy for CI that gives realistic test fidelity without ever moving regulated PII into build environments.](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07068_Data_governance_meets_CI_CD_pipelines_routinely_provision_te.md>)
- [Expert] [How would you design a delivery system where every deployable artifact carries a cryptographically verifiable graph of exactly which source, dependencies, and build steps produced it, and why does that graph matter more than a single signature?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07164_How_would_you_design_a_delivery_system_where_every_deployabl.md>)
- [Expert] [A staff engineer proposes generating pipeline YAML and deploy manifests with an LLM agent to cut boilerplate across 400 services; as the accountable architect, what guardrails, provenance, and review model would you demand before letting AI-authored config touch prod, and where would you refuse it entirely?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07496_A_staff_engineer_proposes_generating_pipeline_YAML_and_deplo.md>)

## Related concepts

- [Securing ML Delivery](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Safe_Config_Rollouts/Securing_ML_Delivery.md>) *(Cloud Fundamentals)*
- [Org-wide Lineage and PII Tracking](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Read-through_and_Write-behind_Caching/Org-wide_Lineage_and_PII_Tracking.md>) *(System Design Core)*
- [Automated Dependency Updates](<../../../../TOOLS/Git_and_GitHub/Git_Core/Automated_Dependency_Updates.md>) *(Git Core)*

