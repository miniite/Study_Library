---
id: "TOOLS/Git_and_GitHub/Git_Core/Rerere_and_Repeated_Conflicts"
level: 4
type: "concept"
topic: "TOOLS/Git_and_GitHub/Git_Core"
tools: ["Git_and_GitHub"]
---

# Rerere and Repeated Conflicts

**L4 Advanced** · [↑ Git Core](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Git and GitHub](<../_index.md>)

## Practice questions

- [Basic] [How do you continue, skip, or abort a rebase that is in progress?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-28119_How_do_you_continue_skip_or_abort_a_rebase_that_is_in_progre.md>)
- [Deep] [A rebase of a 25-commit schema branch onto main keeps re-surfacing the same conflict on a generated schema-dump file every commit. Diagnose why and decide between rerere, a merge driver that regenerates the dump, or squashing before rebase.](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27805_A_rebase_of_a_25-commit_schema_branch_onto_main_keeps_re-sur.md>)
- [Deep] [A rebase of a long-lived infra feature branch onto main keeps re-conflicting on the same generated lockfile every commit. Diagnose why, and decide between rerere, a merge driver that regenerates the lockfile, or squashing before rebase.](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27056_A_rebase_of_a_long-lived_infra_feature_branch_onto_main_keep.md>)
- [Deep] [A rebase of a 30-commit agent-refactor branch keeps re-surfacing the same conflict on a generated tool-schema file every commit. Diagnose why and decide between rerere, a regen merge driver, or squash-first.](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27391_A_rebase_of_a_30-commit_agent-refactor_branch_keeps_re-surfa.md>)
- [Deep] [A rebase of a 30-commit anomaly-detector refactor branch keeps re-surfacing the same conflict on a generated baseline-thresholds file every commit. Diagnose why, and decide between rerere, a merge driver that regenerates the thresholds, or squashing before the rebase.](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27630_A_rebase_of_a_30-commit_anomaly-detector_refactor_branch_kee.md>)
- [Deep] [A rebase of a 40-commit pipeline-refactor branch keeps re-surfacing the same conflict on a generated schema.yml every commit. Diagnose why, and decide between rerere, a merge driver that regenerates the file, or squashing before the rebase.](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-28018_A_rebase_of_a_40-commit_pipeline-refactor_branch_keeps_re-su.md>)
- [Deep] [You keep resolving the identical conflict every time you rebase a long-lived branch onto main. How does rerere help, how do you enable it, and what are its failure modes on subtle conflicts?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-28261_You_keep_resolving_the_identical_conflict_every_time_you_reb.md>)
- [Deep] [You keep resolving the identical conflict every rebase of a long-lived integration branch onto main. Explain how rerere records and replays resolutions, how to enable it repo-wide in CI, and its failure modes on subtly different conflicts.](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-28365_You_keep_resolving_the_identical_conflict_every_rebase_of_a_.md>)

## Related concepts

- [Resilience to Git Outages](<../../../10_DevOps_and_Platform/Delivery/GitOps/Reconciliation_Failures/Resilience_to_Git_Outages.md>) *(GitOps)*
- [Pipeline Rules](<../../GitLab/GitLab_CI/Pipeline_Rules.md>) *(GitLab CI)*
- [Pull Request Pipelines](<../../../10_DevOps_and_Platform/Delivery/CI_CD/Pull_Request_Pipelines.md>) *(CI CD)*

