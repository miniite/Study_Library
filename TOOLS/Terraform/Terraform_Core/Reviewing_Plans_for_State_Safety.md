---
id: "TOOLS/Terraform/Terraform_Core/Reviewing_Plans_for_State_Safety"
level: 4
type: "concept"
topic: "TOOLS/Terraform/Terraform_Core"
tools: ["AWS", "Terraform"]
---

# Reviewing Plans for State Safety

**L4 Advanced** · [↑ Terraform Core](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [AWS](<../../AWS/_index.md>)
- [Terraform](<../_index.md>)

## Practice questions

- [Intermediate] [How do you review a Terraform PR for state safety? What patterns in a diff make you suspect an unintended destroy or replace?](<../../../PRACTICE/TOOLS/Terraform/Terraform_Core/Q-43100_How_do_you_review_a_Terraform_PR_for_state_safety_What_patte.md>)
- [Deep] [Walk me through how you'd migrate a live production state file from local backend to an S3 remote backend with zero drift and no downtime.](<../../../PRACTICE/TOOLS/Terraform/Terraform_Core/Q-43057_Walk_me_through_how_you_d_migrate_a_live_production_state_fi.md>)
- [Deep] [Why is terraform refresh (now baked into plan) both useful and dangerous in a large environment, and how do you control it?](<../../../PRACTICE/TOOLS/Terraform/Terraform_Core/Q-43069_Why_is_terraform_refresh_now_baked_into_plan_both_useful_and.md>)
- [Deep] [Your CI pipeline runs plan on PR and apply on merge. How do you make sure the applied plan matches the reviewed plan given state may have changed in between?](<../../../PRACTICE/TOOLS/Terraform/Terraform_Core/Q-43079_Your_CI_pipeline_runs_plan_on_PR_and_apply_on_merge_How_do_y.md>)
- [Deep] [How do you keep state files consistent when part of your infra is in Terraform Cloud and part is in a self-managed S3 backend?](<../../../PRACTICE/TOOLS/Terraform/Terraform_Core/Q-43098_How_do_you_keep_state_files_consistent_when_part_of_your_inf.md>)
- [Deep] [A colleague ran terraform destroy in the wrong workspace and wiped prod. What guardrails (backend policy, CI gates, prevent_destroy) would have stopped it?](<../../../PRACTICE/TOOLS/Terraform/Terraform_Core/Q-43116_A_colleague_ran_terraform_destroy_in_the_wrong_workspace_and.md>)
- [Deep] [Explain how you'd give an on-call engineer emergency apply access without them being able to read every secret in state day-to-day.](<../../../PRACTICE/TOOLS/Terraform/Terraform_Core/Q-43134_Explain_how_you_d_give_an_on-call_engineer_emergency_apply_a.md>)
- [Deep] [A cross-account assume-role expired mid-apply and Terraform left resources partially created and partly in state. How do you safely resume?](<../../../PRACTICE/TOOLS/Terraform/Terraform_Core/Q-43150_A_cross-account_assume-role_expired_mid-apply_and_Terraform_.md>)

## Related concepts

- [Safe Config Rollouts](<../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Safe_Config_Rollouts/_index.md>) *(Cloud Fundamentals)*
- [Expand and Contract Migrations](<../../../10_DevOps_and_Platform/Delivery/CI_CD/Idempotent_Deploy_Jobs/Expand_and_Contract_Migrations.md>) *(CI CD)*
- [Idempotent Deploy Jobs](<../../../10_DevOps_and_Platform/Delivery/CI_CD/Idempotent_Deploy_Jobs/_index.md>) *(CI CD)*

