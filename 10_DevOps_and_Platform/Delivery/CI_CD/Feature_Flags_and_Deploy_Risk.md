---
id: "10_DevOps_and_Platform/Delivery/CI_CD/Feature_Flags_and_Deploy_Risk"
level: 4
type: "concept"
topic: "10_DevOps_and_Platform/Delivery/CI_CD"
tools: []
---

# Feature Flags and Deploy Risk

**L4 Advanced** · [↑ CI CD](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Practice questions

- [Basic] [What is a feature flag and how can it let you deploy code without turning the feature on?](<../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07343_What_is_a_feature_flag_and_how_can_it_let_you_deploy_code_wi.md>)
- [Intermediate] [You have long-lived feature flags nobody knows are safe to remove; how would you build a lifecycle and a pipeline check that forces flag cleanup without breaking anything still depending on them?](<../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07471_You_have_long-lived_feature_flags_nobody_knows_are_safe_to_r.md>)
- [Deep] [What is a feature flag and how does it decouple deployment from release?](<../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07193_What_is_a_feature_flag_and_how_does_it_decouple_deployment_f.md>)
- [Deep] [How would you deploy a change to your anomaly-scoring service behind a feature flag so you can score in shadow, compare to the live path, and flip only when confident?](<../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06638_How_would_you_deploy_a_change_to_your_anomaly-scoring_servic.md>)
- [Deep] [Long-lived feature flags have piled up and nobody knows which are safe to remove; how do you build a lifecycle and pipeline check that forces flag cleanup?](<../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06881_Long-lived_feature_flags_have_piled_up_and_nobody_knows_whic.md>)
- [Deep] [A deploy that flips a feature flag caused a cascading load increase downstream that the flag owner didn't anticipate; how would you build flag changes into the same risk-gated, observable pipeline flow as code deploys?](<../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07019_A_deploy_that_flips_a_feature_flag_caused_a_cascading_load_i.md>)
- [Deep] [You're asked whether the org should treat feature-flag changes as deploys subject to the same gating and audit; make the architectural argument for unifying flag and deploy governance and what breaks if they stay separate.](<../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07187_You_re_asked_whether_the_org_should_treat_feature-flag_chang.md>)
- [Expert] [You're setting the org standard for how feature flags relate to deploys; teams are drowning in stale flags and inconsistent flag behavior across services. Design the flag-lifecycle governance and platform tooling that keeps flags fast, consistent, and reliably cleaned up at scale.](<../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07121_You_re_setting_the_org_standard_for_how_feature_flags_relate.md>)

## Related concepts

- [Correlated Failure Prevention](<../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Safe_Config_Rollouts/Correlated_Failure_Prevention.md>) *(Cloud Fundamentals)*
- [Org-wide Lineage and PII Tracking](<../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Read-through_and_Write-behind_Caching/Org-wide_Lineage_and_PII_Tracking.md>) *(System Design Core)*
- [Sync Notifications](<../GitOps/GitOps_Fundamentals/Sync_Notifications.md>) *(GitOps)*

