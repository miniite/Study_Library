---
id: "10_DevOps_and_Platform/Delivery/CI_CD/Push_vs_Pull_Deployments"
level: 5
type: "deep-dive"
topic: "10_DevOps_and_Platform/Delivery/CI_CD"
tools: ["Kubernetes", "Git_and_GitHub", "Argo_CD"]
---

# Push vs Pull Deployments

**L5 Expert** · [↑ CI CD](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Kubernetes](<../../../../TOOLS/Kubernetes/_index.md>)
- [Git and GitHub](<../../../../TOOLS/Git_and_GitHub/_index.md>)
- [Argo CD](<../../../../TOOLS/Argo_CD/_index.md>)

## Practice questions

- [Basic] [What is the role of git as the 'source of truth' for your alerting setup, and what problem does that solve?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06488_What_is_the_role_of_git_as_the_source_of_truth_for_your_aler.md>)
- [Deep] [What is GitOps and how does it change the way deployments are done?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07259_What_is_GitOps_and_how_does_it_change_the_way_deployments_ar.md>)
- [Deep] [How would you deploy DAGs to Airflow via GitOps (git-sync or object-storage bundle) rather than baking them into the image, and what are the trade-offs at deploy time?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06658_How_would_you_deploy_DAGs_to_Airflow_via_GitOps_git-sync_or_.md>)
- [Deep] [You run deploys via ArgoCD but a sync keeps flapping between healthy and progressing on one service; walk me through how you'd debug whether it's a bad health check, a mutating webhook, or resource-hash instability.](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07384_You_run_deploys_via_ArgoCD_but_a_sync_keeps_flapping_between.md>)
- [Expert] [With GitOps, a manual kubectl edit drifts prod away from the repo; how does the reconciliation loop react, and how do you make drift visible and auditable?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06854_With_GitOps_a_manual_kubectl_edit_drifts_prod_away_from_the_.md>)
- [Expert] [Your service is deployed by GitOps but an urgent incident needs a change faster than the reconcile loop and PR flow allow; how do you design a safe break-glass path that stays auditable and reconciles back?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07042_Your_service_is_deployed_by_GitOps_but_an_urgent_incident_ne.md>)
- [Expert] [How would you decide whether deploy orchestration should be pull-based (GitOps reconciler) or push-based for a fleet that spans managed Kubernetes, VMs, and serverless simultaneously, and can one model actually serve all three?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07173_How_would_you_decide_whether_deploy_orchestration_should_be_.md>)
- [Expert] [Your org is adopting Kubernetes multi-cluster with a fleet-management layer; how do you decide the split of responsibility between the fleet controller, the per-cluster GitOps agent, and the CD pipeline for who declares intended version, who reconciles, and who owns rollback?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07518_Your_org_is_adopting_Kubernetes_multi-cluster_with_a_fleet-m.md>)

## Related concepts

- [GitOps Fundamentals](<../../GitOps/GitOps_Fundamentals/_index.md>) *(GitOps)*
- [Multi-cluster Ordering](<../../GitOps/GitOps_Fundamentals/Multi-cluster_Ordering.md>) *(GitOps)*
- [Resilience to Git Outages](<../../GitOps/Reconciliation_Failures/Resilience_to_Git_Outages.md>) *(GitOps)*

