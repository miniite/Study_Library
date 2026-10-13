---
id: "10_DevOps_and_Platform/Delivery/GitOps/Resilience_to_Git_Outages"
level: 5
type: "deep-dive"
topic: "10_DevOps_and_Platform/Delivery/GitOps"
tools: ["Git_and_GitHub", "Kubernetes", "Argo_CD", "Docker"]
---

# Resilience to Git Outages

**L5 Expert** · [↑ GitOps](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Git and GitHub](<../../../../TOOLS/Git_and_GitHub/_index.md>)
- [Kubernetes](<../../../../TOOLS/Kubernetes/_index.md>)
- [Argo CD](<../../../../TOOLS/Argo_CD/_index.md>)
- [Docker](<../../../../TOOLS/Docker/_index.md>)

## Practice questions

- [Basic] [What does 'reconciliation' mean in the context of GitOps?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/GitOps/Q-29090_What_does_reconciliation_mean_in_the_context_of_GitOps.md>)
- [Intermediate] [How would you validate manifests before they are merged into a GitOps repo?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/GitOps/Q-29154_How_would_you_validate_manifests_before_they_are_merged_into.md>)
- [Deep] [A GitOps rollback by reverting the commit didn't restore service because the old image tag had been garbage-collected from the registry — how do you make rollbacks truly reliable?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/GitOps/Q-29215_A_GitOps_rollback_by_reverting_the_commit_didn_t_restore_ser.md>)
- [Deep] [Your image automation opened a PR but the image scan failed after merge — how do you fail the deploy safely and prevent the vulnerable image from reconciling?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/GitOps/Q-29236_Your_image_automation_opened_a_PR_but_the_image_scan_failed_.md>)
- [Deep] [A team wants feature-branch preview environments spun up per PR via GitOps — how do you generate and garbage-collect ephemeral environments cleanly?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/GitOps/Q-29265_A_team_wants_feature-branch_preview_environments_spun_up_per.md>)
- [Deep] [How do you set the org policy on pruning stateful resources so an accidental manifest deletion can never destroy a PVC or database, while keeping true declarative cleanup for stateless apps?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/GitOps/Q-29334_How_do_you_set_the_org_policy_on_pruning_stateful_resources_.md>)
- [Deep] [How would you quantify and communicate the residual risk that git is a single source of truth AND a single point of failure — what mitigations make that acceptable at org scale?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/GitOps/Q-29376_How_would_you_quantify_and_communicate_the_residual_risk_tha.md>)
- [Expert] [Your GitOps repo webhooks and CI both mutate the same branch, causing race conditions on concurrent image bumps — how do you serialize commits to avoid lost updates?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/GitOps/Q-29281_Your_GitOps_repo_webhooks_and_CI_both_mutate_the_same_branch.md>)

## Related concepts

- [Push vs Pull Deployments](<../../CI_CD/Deployment_Observability/Push_vs_Pull_Deployments.md>) *(CI CD)*
- [Reverts in GitOps Rollouts](<../../../../TOOLS/Git_and_GitHub/Git_Core/Reverts_in_GitOps_Rollouts.md>) *(Git Core)*
- [Protected Branches and Required Checks](<../../../../TOOLS/Git_and_GitHub/Git_Core/Protected_Branches_and_Required_Checks.md>) *(Git Core)*

