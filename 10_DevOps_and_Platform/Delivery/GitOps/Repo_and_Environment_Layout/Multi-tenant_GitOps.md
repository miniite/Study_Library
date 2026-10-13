---
id: "10_DevOps_and_Platform/Delivery/GitOps/Multi-tenant_GitOps"
level: 5
type: "deep-dive"
topic: "10_DevOps_and_Platform/Delivery/GitOps"
tools: ["Kubernetes", "Git_and_GitHub", "Argo_CD"]
---

# Multi-tenant GitOps

**L5 Expert** · [↑ GitOps](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Kubernetes](<../../../../TOOLS/Kubernetes/_index.md>)
- [Git and GitHub](<../../../../TOOLS/Git_and_GitHub/_index.md>)
- [Argo CD](<../../../../TOOLS/Argo_CD/_index.md>)

## Practice questions

- [Intermediate] [What is RBAC and how does it relate to controlling who can deploy in GitOps?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/GitOps/Q-29129_What_is_RBAC_and_how_does_it_relate_to_controlling_who_can_d.md>)
- [Deep] [How would you use ApplicationSet with a Git-directory generator to onboard new services automatically while still enforcing per-team RBAC?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/GitOps/Q-29213_How_would_you_use_ApplicationSet_with_a_Git-directory_genera.md>)
- [Deep] [You must give 12 dev teams self-service deploys via GitOps without letting them touch each other's namespaces — design the Argo CD Project and repo permissions.](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/GitOps/Q-29214_You_must_give_12_dev_teams_self-service_deploys_via_GitOps_w.md>)
- [Deep] [How do you architect GitOps for a multi-tenant SaaS where each customer gets an isolated namespace and config, but the base platform is shared?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/GitOps/Q-29222_How_do_you_architect_GitOps_for_a_multi-tenant_SaaS_where_ea.md>)
- [Deep] [Design a governance model for who can approve prod merges across 40 teams — CODEOWNERS, branch protection, quorum reviews — that scales without becoming an approval bottleneck.](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/GitOps/Q-29321_Design_a_governance_model_for_who_can_approve_prod_merges_ac.md>)
- [Deep] [How would you architect dual-ownership resolution so two controllers (Argo CD and a Helm operator) can never end up fighting over the same resource across the fleet?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/GitOps/Q-29344_How_would_you_architect_dual-ownership_resolution_so_two_con.md>)
- [Expert] [Argue the trade-off between a single central Argo CD control plane for the whole org versus per-BU instances — governance, cost, blast radius, and operational toil each pull differently.](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/GitOps/Q-29291_Argue_the_trade-off_between_a_single_central_Argo_CD_control.md>)
- [Expert] [How would you unify two acquired companies each running a different GitOps stack (one Flux, one Argo CD) into a single governed platform without freezing delivery for either?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/GitOps/Q-29294_How_would_you_unify_two_acquired_companies_each_running_a_di.md>)

## Related concepts

- [Push vs Pull Deployments](<../../CI_CD/Deployment_Observability/Push_vs_Pull_Deployments.md>) *(CI CD)*
- [Org-wide Lineage and PII Tracking](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Read-through_and_Write-behind_Caching/Org-wide_Lineage_and_PII_Tracking.md>) *(System Design Core)*
- [API Composition and BFF](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Idempotent_Order_APIs/API_Composition_and_BFF.md>) *(System Design Core)*

