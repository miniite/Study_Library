---
id: "TOOLS/Git_and_GitHub/Git_Core/Reverting_Merges"
level: 4
type: "concept"
topic: "TOOLS/Git_and_GitHub/Git_Core"
tools: ["Git_and_GitHub"]
---

# Reverting Merges

**L4 Advanced** · [↑ Git Core](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Git and GitHub](<../_index.md>)

## Practice questions

- [Intermediate] [How does git store a commit's parent, and how does that make history a directed acyclic graph?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-28132_How_does_git_store_a_commit_s_parent_and_how_does_that_make_.md>)
- [Intermediate] [Your Airflow DAGs are deployed by syncing a Git repo to the scheduler, and a merged PR renamed a DAG file, orphaning all historical task instances and losing the schedule state. Why does renaming a DAG break history, and how would you gate DAG-id-affecting changes in a PR check?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27981_Your_Airflow_DAGs_are_deployed_by_syncing_a_Git_repo_to_the_.md>)
- [Deep] [After a squash-merge of a big schema feature, teammates who branched off the pre-squash work keep re-introducing already-applied migrations into their branches. Explain the DAG reason and the exact workflow to reset their branches cleanly onto main.](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27797_After_a_squash-merge_of_a_big_schema_feature_teammates_who_b.md>)
- [Deep] [A revert of a bad migration merge to main is blocking the later re-merge of the corrected feature because Git thinks the changes are already applied. Explain the DAG mechanism and the exact sequence to cleanly re-introduce the migration.](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27812_A_revert_of_a_bad_migration_merge_to_main_is_blocking_the_la.md>)
- [Deep] [After a squash-merge of a model-feature branch, teammates who branched off the pre-squash work keep re-introducing 'already merged' commits into retrains. Explain the DAG reason and give the exact workflow to reset their branches cleanly onto main.](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27212_After_a_squash-merge_of_a_model-feature_branch_teammates_who.md>)
- [Deep] [A revert of a bad feature-engineering merge to main is blocking the later re-merge of the same feature because Git thinks it's already applied. Explain the mechanism at the DAG level and the exact sequence to cleanly re-introduce it in the training repo.](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27248_A_revert_of_a_bad_feature-engineering_merge_to_main_is_block.md>)
- [Deep] [After a squash-merge of a big RAG feature, teammates who branched off the pre-squash branch keep re-introducing 'already merged' prompt commits. Explain the DAG reason and the workflow to reset their branches cleanly.](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27405_After_a_squash-merge_of_a_big_RAG_feature_teammates_who_bran.md>)
- [Deep] [A revert of a bad alert-routing merge to main is blocking the later re-merge of the same correlation improvement because Git thinks it's already applied. Explain the DAG mechanism and the exact sequence to cleanly re-introduce the correlation change into your release train.](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27602_A_revert_of_a_bad_alert-routing_merge_to_main_is_blocking_th.md>)

## Related concepts

- [Resilience to Git Outages](<../../../10_DevOps_and_Platform/Delivery/GitOps/Reconciliation_Failures/Resilience_to_Git_Outages.md>) *(GitOps)*
- [Pull Request Pipelines](<../../../10_DevOps_and_Platform/Delivery/CI_CD/Pull_Request_Pipelines.md>) *(CI CD)*
- [Pipeline Rules](<../../GitLab/GitLab_CI/Pipeline_Rules.md>) *(GitLab CI)*

