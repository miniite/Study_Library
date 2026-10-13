---
id: "TOOLS/Git_and_GitHub/Git_Core/Reverts_in_GitOps_Rollouts"
level: 4
type: "concept"
topic: "TOOLS/Git_and_GitHub/Git_Core"
tools: ["Git_and_GitHub", "Kubernetes"]
---

# Reverts in GitOps Rollouts

**L4 Advanced** · [↑ Git Core](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Git and GitHub](<../_index.md>)
- [Kubernetes](<../../Kubernetes/_index.md>)

## Practice questions

- [Intermediate] [How would you roll back a bad Kubernetes deployment when your manifests are managed in Git via GitOps?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-28474_How_would_you_roll_back_a_bad_Kubernetes_deployment_when_you.md>)
- [Deep] [An outage was caused by a manifest that passed CI policy but violated an unwritten operational rule, and Git shows the change but not the reasoning. How would you enforce that risky infra PRs carry a documented rollout plan and rollback linked in the commit or PR?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27052_An_outage_was_caused_by_a_manifest_that_passed_CI_policy_but.md>)
- [Deep] [A canary rollout of a new model reads its traffic-split config from Git, but a mid-rollout revert to the previous commit left canary and stable inconsistent. How does a Git revert interact with an in-flight progressive rollout, and how do you abort cleanly?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27252_A_canary_rollout_of_a_new_model_reads_its_traffic-split_conf.md>)
- [Deep] [You want to roll back a fine-tuned model to a prior adapter version if the new one regresses on evals. How would you tie the adapter artifact to a Git tag so the rollback is one deterministic step?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27379_You_want_to_roll_back_a_fine-tuned_model_to_a_prior_adapter_.md>)
- [Deep] [You need to roll out a schema-affecting change across backend + frontend in lockstep so neither deploys ahead of the other. How do you use Git branching, PR dependencies, and deploy gating to coordinate?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-28297_You_need_to_roll_out_a_schema-affecting_change_across_backen.md>)
- [Expert] [You need to revert a feature in production, but its Git commit bundled application code with an already-applied irreversible data migration. How do you reason about what git revert can and cannot undo here, and what compensating migration would you author?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27784_You_need_to_revert_a_feature_in_production_but_its_Git_commi.md>)
- [Expert] [A rollback to an older prompt+model commit redeployed an old agent, but the vector index had already migrated to a new embedding space, breaking retrieval. How do you couple Git-driven rollbacks with index state so a code revert doesn't fight the index?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27455_A_rollback_to_an_older_prompt_model_commit_redeployed_an_old.md>)
- [Expert] [Your GitOps-managed alerting stack has a 3-minute reconcile interval, so a git revert of a bad routing change isn't fast enough during a paging flood. How do you reconcile the Git-as-source-of-truth invariant with the need for a sub-minute suppression during an active incident?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27598_Your_GitOps-managed_alerting_stack_has_a_3-minute_reconcile_.md>)

## Related concepts

- [Resilience to Git Outages](<../../../10_DevOps_and_Platform/Delivery/GitOps/Reconciliation_Failures/Resilience_to_Git_Outages.md>) *(GitOps)*
- [Rollback Strategy](<../../../10_DevOps_and_Platform/Delivery/CI_CD/Rollback_Strategy/_index.md>) *(CI CD)*
- [Push vs Pull Deployments](<../../../10_DevOps_and_Platform/Delivery/CI_CD/Deployment_Observability/Push_vs_Pull_Deployments.md>) *(CI CD)*

