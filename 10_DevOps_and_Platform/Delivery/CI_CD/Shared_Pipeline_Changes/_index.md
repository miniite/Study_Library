---
id: "10_DevOps_and_Platform/Delivery/CI_CD/Shared_Pipeline_Changes"
level: 4
type: "concept"
topic: "10_DevOps_and_Platform/Delivery/CI_CD"
tools: ["Git_and_GitHub", "Docker", "GitLab"]
---

# Shared Pipeline Changes

**L4 Advanced** · [↑ CI CD](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Go deeper

- [Schema Evolution in Event Driven Deploys](<Schema_Evolution_in_Event_Driven_Deploys.md>)
- [Deployment Gating Trade-offs](<Deployment_Gating_Trade-offs.md>)
- [Safe Delivery Platform Changes](<Safe_Delivery_Platform_Changes.md>)
- [Golden Pipelines and Templates](<Golden_Pipelines_and_Templates.md>)

## Implementations (tools)

- [Git and GitHub](<../../../../TOOLS/Git_and_GitHub/_index.md>)
- [Docker](<../../../../TOOLS/Docker/_index.md>)
- [GitLab](<../../../../TOOLS/GitLab/_index.md>)

## Practice questions

- [Basic] [Why do teams keep dashboards, alerts, and runbooks together in one git repo and ship them through the same pipeline?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06438_Why_do_teams_keep_dashboards_alerts_and_runbooks_together_in.md>)
- [Intermediate] [Your pipeline builds one artifact but three teams consume it with different feature flag defaults; how do you keep the artifact truly immutable while letting per-environment config diverge safely?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06947_Your_pipeline_builds_one_artifact_but_three_teams_consume_it.md>)
- [Deep] [How would you structure a monorepo pipeline for an AI product where a change to the prompt package, the retriever package, or the model config each triggers only the relevant evals?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06364_How_would_you_structure_a_monorepo_pipeline_for_an_AI_produc.md>)
- [Deep] [How would you structure a monorepo pipeline so a change to a shared feature library re-tests exactly the models that consume it and nothing else?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07656_How_would_you_structure_a_monorepo_pipeline_so_a_change_to_a.md>)
- [Deep] [Explain how you'd version and deploy a gRPC service so that adding a field, deprecating a field, and removing a field each map to distinct, safe pipeline stages across producer and consumers.](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06946_Explain_how_you_d_version_and_deploy_a_gRPC_service_so_that_.md>)
- [Deep] [Two teams' pipelines both apply Terraform to overlapping infrastructure and keep fighting over the same resources; how would you carve state and ownership so their deploys stop stepping on each other?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07006_Two_teams_pipelines_both_apply_Terraform_to_overlapping_infr.md>)
- [Deep] [A deploy of a shared library version must reach fifty consumer services, but you want the library team accountable for breakage, not the consumers; how do you architect the responsibility and the gating so incentives align?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07172_A_deploy_of_a_shared_library_version_must_reach_fifty_consum.md>)
- [Expert] [How would you build a deploy pipeline for a service where each tenant can be on a different version (multi-version production), and how do you keep the shared database compatible across all live versions?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07003_How_would_you_build_a_deploy_pipeline_for_a_service_where_ea.md>)

## Related concepts

- [Securing ML Delivery](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Safe_Config_Rollouts/Securing_ML_Delivery.md>) *(Cloud Fundamentals)*
- [Monorepo CI Optimization](<../../../../TOOLS/Git_and_GitHub/Git_Core/Monorepo_CI_Optimization.md>) *(Git Core)*
- [Coordinated Client Upgrades](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Read-through_and_Write-behind_Caching/Coordinated_Client_Upgrades.md>) *(System Design Core)*

