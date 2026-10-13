---
id: "TOOLS/Git_and_GitHub/Git_Core/Recovering_Lost_Commits"
level: 4
type: "concept"
topic: "TOOLS/Git_and_GitHub/Git_Core"
tools: ["Git_and_GitHub"]
---

# Recovering Lost Commits

**L4 Advanced** · [↑ Git Core](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Git and GitHub](<../_index.md>)

## Practice questions

- [Basic] [How would you delete a stale experiment branch like alter-orders-v2 both locally and on GitHub after its migration was merged?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27730_How_would_you_delete_a_stale_experiment_branch_like_alter-or.md>)
- [Intermediate] [How would you recover an earlier version of your alertmanager config after you overwrote it and committed a worse version three commits ago?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27502_How_would_you_recover_an_earlier_version_of_your_alertmanage.md>)
- [Intermediate] [A junior ran git reset --hard on the alert-correlation branch and thinks a day of tuning work is gone. Walk through recovery via reflog and fsck --lost-found, and the exact conditions under which those correlation-rule commits are truly unrecoverable.](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27649_A_junior_ran_git_reset_--hard_on_the_alert-correlation_branc.md>)
- [Intermediate] [A stale ArgoCD Application still points at a deleted Git branch and shows Unknown, blocking your dashboard's health view. How would you audit and garbage-collect Applications whose source refs no longer exist, and automate that hygiene?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-28614_A_stale_ArgoCD_Application_still_points_at_a_deleted_Git_bra.md>)
- [Deep] [A teammate force-pushed a shared long-lived model-integration branch and three engineers' experiment commits vanished from the tip. As on-call owner, recover the lost commits and design guardrails so it can't happen again.](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27221_A_teammate_force-pushed_a_shared_long-lived_model-integratio.md>)
- [Deep] [A teammate force-pushed a shared long-lived alert-tuning branch and three engineers' suppression-rule commits vanished from the tip right before an on-call rotation. As on-call owner, recover the lost commits with reflog and design guardrails so it can't recur.](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27651_A_teammate_force-pushed_a_shared_long-lived_alert-tuning_bra.md>)
- [Deep] [A rebase went wrong halfway and the branch is in a weird half-applied state with the wrong commits. Walk through diagnosing the state and getting back to the exact pre-rebase HEAD using reflog and ORIG_HEAD.](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-28260_A_rebase_went_wrong_halfway_and_the_branch_is_in_a_weird_hal.md>)
- [Expert] [You committed a secret and pushed it. What steps do you take to remediate?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-28666_You_committed_a_secret_and_pushed_it_What_steps_do_you_take_.md>)

## Related concepts

- [Resilience to Git Outages](<../../../10_DevOps_and_Platform/Delivery/GitOps/Reconciliation_Failures/Resilience_to_Git_Outages.md>) *(GitOps)*
- [Pinning Third-party Actions](<../GitHub_Actions/Pinning_Third-party_Actions.md>) *(GitHub Actions)*
- [Reconciliation Failures](<../../../10_DevOps_and_Platform/Delivery/GitOps/Reconciliation_Failures/_index.md>) *(GitOps)*

