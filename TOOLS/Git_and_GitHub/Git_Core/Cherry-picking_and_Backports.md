---
id: "TOOLS/Git_and_GitHub/Git_Core/Cherry-picking_and_Backports"
level: 4
type: "concept"
topic: "TOOLS/Git_and_GitHub/Git_Core"
tools: ["Git_and_GitHub"]
---

# Cherry-picking and Backports

**L4 Advanced** · [↑ Git Core](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Git and GitHub](<../_index.md>)

## Practice questions

- [Intermediate] [An urgent index-fix commit needs to go onto a release branch without dragging over other unmerged schema work. Walk through using git cherry-pick, why -x matters for the audit trail, and how you'd handle a conflict on the migration-version numbering.](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27817_An_urgent_index-fix_commit_needs_to_go_onto_a_release_branch.md>)
- [Intermediate] [How would you use git cherry-pick to apply a single urgent fix to a source-parsing bug onto your release branch without bringing over other in-progress changes?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27950_How_would_you_use_git_cherry-pick_to_apply_a_single_urgent_f.md>)
- [Intermediate] [A production hotfix needs to go to main immediately but main is behind an in-progress big feature merge. How do you branch, fix, and land it safely without dragging the unfinished feature into prod?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-28229_A_production_hotfix_needs_to_go_to_main_immediately_but_main.md>)
- [Deep] [A hotfix index needs to reach the current release branch, main, and two older LTS release branches whose schema diverged. Compare cherry-pick -x, rebase --onto, and format-patch/am for landing the migration cleanly on each and preserving provenance.](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27787_A_hotfix_index_needs_to_reach_the_current_release_branch_mai.md>)
- [Deep] [You need to back-port a data-correctness fix to a stored procedure across five older release tags where the surrounding package diverged. Walk through cherry-pick -x for provenance, rebase --onto for a range, and format-patch/am for offline transfer, and how you resolve per-tag conflicts.](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27873_You_need_to_back-port_a_data-correctness_fix_to_a_stored_pro.md>)
- [Deep] [Two release branches of the serving stack both need the same model-safety patch but the surrounding code diverged. Compare cherry-pick -x, rebase --onto, and format-patch/am for landing it on each, and when each is preferable for provenance.](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27226_Two_release_branches_of_the_serving_stack_both_need_the_same.md>)
- [Deep] [Two release branches of your model-serving config both need the same latency-optimization fix but the surrounding code diverged. Compare cherry-pick, merge-forward, and a shared base-branch approach for keeping them in sync.](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27431_Two_release_branches_of_your_model-serving_config_both_need_.md>)
- [Deep] [A production data hotfix (fix a bad join corrupting revenue numbers) must land on main immediately, but main is mid-way through a large lakehouse-migration refactor that isn't ready. How do you branch from the last good tag, land the fix, and avoid dragging the refactor into prod?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-28030_A_production_data_hotfix_fix_a_bad_join_corrupting_revenue_n.md>)

## Related concepts

- [Resilience to Git Outages](<../../../10_DevOps_and_Platform/Delivery/GitOps/Reconciliation_Failures/Resilience_to_Git_Outages.md>) *(GitOps)*
- [Pinning Third-party Actions](<../GitHub_Actions/Pinning_Third-party_Actions.md>) *(GitHub Actions)*
- [Pull Request Pipelines](<../../../10_DevOps_and_Platform/Delivery/CI_CD/Pull_Request_Pipelines.md>) *(CI CD)*

