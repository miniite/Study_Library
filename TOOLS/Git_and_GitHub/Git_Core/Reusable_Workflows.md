---
id: "TOOLS/Git_and_GitHub/Git_Core/Reusable_Workflows"
level: 4
type: "concept"
topic: "TOOLS/Git_and_GitHub/Git_Core"
tools: ["Git_and_GitHub", "Python", "dbt"]
---

# Reusable Workflows

**L4 Advanced** · [↑ Git Core](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Git and GitHub](<../_index.md>)
- [Python](<../../Python/_index.md>)
- [dbt](<../../dbt/_index.md>)

## Practice questions

- [Intermediate] [You maintain a design-system package consumed by five apps. Walk through your release flow using tags, semantic versioning, and a GitHub Actions publish job, and how you'd handle a broken release.](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-28247_You_maintain_a_design-system_package_consumed_by_five_apps_W.md>)
- [Deep] [What is a reusable workflow (or shared pipeline template), and why would a platform team maintain one centrally?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-28525_What_is_a_reusable_workflow_or_shared_pipeline_template_and_.md>)
- [Deep] [A reusable training workflow shared across all model repos was updated and silently broke retrains in repos pinned to @main. How do you version reusable workflows so a central change can't break pinned consumers, and roll a fix org-wide?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27199_A_reusable_training_workflow_shared_across_all_model_repos_w.md>)
- [Deep] [You maintain a GenAI SDK consumed by five internal apps. Walk through your release flow using annotated tags, semver, and a GitHub Actions publish job, and how you'd yank a broken release that emits unsafe output.](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27397_You_maintain_a_GenAI_SDK_consumed_by_five_internal_apps_Walk.md>)
- [Deep] [You maintain a shared alert-correlation-rules library consumed by eight service teams via a Git ref, and a breaking correlation-group change reached all of them because consumers tracked main. Design a versioning and pinning scheme so a rules change can't silently fan out to every team's paging.](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27582_You_maintain_a_shared_alert-correlation-rules_library_consum.md>)
- [Deep] [A reusable alert-validation workflow shared across all observability repos was updated and silently broke rule deploys in repos pinned to @main. How do you version reusable workflows so a central change can't break pinned consumers, and how do you roll a fix org-wide?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27671_A_reusable_alert-validation_workflow_shared_across_all_obser.md>)
- [Deep] [Your org wants an internal shared GitHub Actions workflow that all repos call, but you must be able to update it centrally without breaking pinned versions. How do you version and consume reusable workflows safely?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-28271_Your_org_wants_an_internal_shared_GitHub_Actions_workflow_th.md>)
- [Deep] [You maintain a shared Ansible-roles repo consumed by 20 playbook repos via a Git requirements file. A breaking role change went out because consumers tracked the main branch. Design a versioning and pinning scheme so a role change can't silently reach every consumer.](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-28559_You_maintain_a_shared_Ansible-roles_repo_consumed_by_20_play.md>)

## Related concepts

- [Pinning Third-party Actions](<../GitHub_Actions/Pinning_Third-party_Actions.md>) *(GitHub Actions)*
- [Build and Release Metadata](<../../../10_DevOps_and_Platform/Delivery/CI_CD/Build_and_Release_Metadata/_index.md>) *(CI CD)*
- [Shared Pipeline Changes](<../../../10_DevOps_and_Platform/Delivery/CI_CD/Shared_Pipeline_Changes/_index.md>) *(CI CD)*

