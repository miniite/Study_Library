---
id: "10_DevOps_and_Platform/Delivery/CI_CD/Emergency_Deploy_Paths"
level: 4
type: "concept"
topic: "10_DevOps_and_Platform/Delivery/CI_CD"
tools: []
---

# Emergency Deploy Paths

**L4 Advanced** · [↑ CI CD](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Practice questions

- [Basic] [Why is it useful for a pipeline to fail fast rather than running every step even after an error?](<../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07308_Why_is_it_useful_for_a_pipeline_to_fail_fast_rather_than_run.md>)
- [Intermediate] [How would you design a fast 'smoke-train' CI stage that catches obvious training breakage in minutes without waiting for a full run?](<../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07636_How_would_you_design_a_fast_smoke-train_CI_stage_that_catche.md>)
- [Intermediate] [A hotfix pipeline skips the full test suite to go fast, but engineers started abusing it for normal changes; how do you technically constrain the fast path so it can only carry genuine hotfixes?](<../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06978_A_hotfix_pipeline_skips_the_full_test_suite_to_go_fast_but_e.md>)
- [Intermediate] [Your deploy freeze during a big sale needs to allow one critical fix through; how would you encode freeze windows in the pipeline and permit controlled, audited exceptions?](<../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07443_Your_deploy_freeze_during_a_big_sale_needs_to_allow_one_crit.md>)
- [Deep] [You need to hotfix production urgently. How do you get it out safely through CI/CD?](<../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07670_You_need_to_hotfix_production_urgently_How_do_you_get_it_out.md>)
- [Deep] [How would you design a break-glass fast deploy for an alerting or remediation hotfix during an active incident that skips slow gates but stays auditable and reconciles back?](<../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06632_How_would_you_design_a_break-glass_fast_deploy_for_an_alerti.md>)
- [Deep] [You need a hotfix to a broken production pipeline urgently, but full CI (clone + data-diff) takes 40 minutes; how would you design a safe expedited path for data pipelines that can't become the default?](<../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06738_You_need_a_hotfix_to_a_broken_production_pipeline_urgently_b.md>)
- [Deep] [How would you enforce a deploy freeze on data pipelines during month-end financial close while still allowing an audited critical fix through?](<../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06740_How_would_you_enforce_a_deploy_freeze_on_data_pipelines_duri.md>)

## Related concepts

- [Securing ML Delivery](<../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Safe_Config_Rollouts/Securing_ML_Delivery.md>) *(Cloud Fundamentals)*
- [Pipeline Design Patterns](<../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Design/Pipeline_Design_Patterns.md>) *(Pipeline Design)*
- [Automated Dependency Updates](<../../../TOOLS/Git_and_GitHub/Git_Core/Automated_Dependency_Updates.md>) *(Git Core)*

