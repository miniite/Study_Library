---
id: "TOOLS/Terraform/Terraform_Core/Refactoring_with_State_Moves"
level: 4
type: "concept"
topic: "TOOLS/Terraform/Terraform_Core"
tools: ["Terraform", "AWS"]
---

# Refactoring with State Moves

**L4 Advanced** · [↑ Terraform Core](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Terraform](<../_index.md>)
- [AWS](<../../AWS/_index.md>)

## Practice questions

- [Intermediate] [Your terraform plan shows a resource being destroyed and recreated that you only renamed in code. How do you use terraform state mv (or moved blocks) to avoid the destroy?](<../../../PRACTICE/TOOLS/Terraform/Terraform_Core/Q-43058_Your_terraform_plan_shows_a_resource_being_destroyed_and_rec.md>)
- [Intermediate] [How do lifecycle meta-arguments like create_before_destroy and prevent_destroy interact with what Terraform records in state?](<../../../PRACTICE/TOOLS/Terraform/Terraform_Core/Q-43074_How_do_lifecycle_meta-arguments_like_create_before_destroy_a.md>)
- [Intermediate] [terraform plan reports 'Objects have changed outside of Terraform.' Walk me through how you decide to accept the drift versus revert it.](<../../../PRACTICE/TOOLS/Terraform/Terraform_Core/Q-43127_terraform_plan_reports_Objects_have_changed_outside_of_Terra.md>)
- [Deep] [How do sensitive outputs and the sensitive flag behave in state versus plan output? Does marking sensitive actually protect the value at rest?](<../../../PRACTICE/TOOLS/Terraform/Terraform_Core/Q-43085_How_do_sensitive_outputs_and_the_sensitive_flag_behave_in_st.md>)
- [Deep] [A resource shows perpetual diff on every plan even with no code change. How do you debug ignore_changes, computed fields, and provider bugs?](<../../../PRACTICE/TOOLS/Terraform/Terraform_Core/Q-43094_A_resource_shows_perpetual_diff_on_every_plan_even_with_no_c.md>)
- [Deep] [You migrated from provider v4 to v5 and the plan wants to change dozens of attributes. How do state schema upgrades and moved/replaced blocks help here?](<../../../PRACTICE/TOOLS/Terraform/Terraform_Core/Q-43101_You_migrated_from_provider_v4_to_v5_and_the_plan_wants_to_ch.md>)
- [Deep] [Explain the trade-off of putting many environments' resources under for_each in one state versus separate state per environment. When does each win?](<../../../PRACTICE/TOOLS/Terraform/Terraform_Core/Q-43129_Explain_the_trade-off_of_putting_many_environments_resources.md>)
- [Deep] [A plan replaces a load balancer because subnet_ids order changed. Why is list order significant here, and how do you make the module order-stable?](<../../../PRACTICE/TOOLS/Terraform/Terraform_Core/Q-43142_A_plan_replaces_a_load_balancer_because_subnet_ids_order_cha.md>)

## Related concepts

- [Safe Config Rollouts](<../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Safe_Config_Rollouts/_index.md>) *(Cloud Fundamentals)*
- [Expand and Contract Migrations](<../../../10_DevOps_and_Platform/Delivery/CI_CD/Idempotent_Deploy_Jobs/Expand_and_Contract_Migrations.md>) *(CI CD)*
- [Azure DevOps and Artifact Governance](<../../Azure/Azure_Platform/Azure_DevOps_and_Artifact_Governance/_index.md>) *(Azure Platform)*

