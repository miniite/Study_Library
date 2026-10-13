---
id: "TOOLS/Git_and_GitHub/Git_Core/Atomic_Commits"
level: 4
type: "concept"
topic: "TOOLS/Git_and_GitHub/Git_Core"
tools: ["Git_and_GitHub", "dbt"]
---

# Atomic Commits

**L4 Advanced** · [↑ Git Core](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Git and GitHub](<../_index.md>)
- [dbt](<../../dbt/_index.md>)

## Practice questions

- [Intermediate] [A PR changed a view definition and a stored procedure that reads it in the same commit, and a reviewer can't tell whether a broken query came from the view or the proc. How would you split concerns across commits so a bad result is bisectable?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27802_A_PR_changed_a_view_definition_and_a_stored_procedure_that_r.md>)
- [Intermediate] [During code review you see a PR mixing a data-loader refactor, a new feature, and a metric-logging fix in one commit. Explain concretely why this hurts revertability and bisect for a production model, and how you'd coach the author to restructure it.](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27234_During_code_review_you_see_a_PR_mixing_a_data-loader_refacto.md>)
- [Intermediate] [During code review you see a PR mixing a log-parser refactor, a new alert rule, and a metric-relabel fix in one commit. Explain concretely why this hurts revertability and bisect for a production monitoring service, and how you'd coach the author to restructure it.](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27660_During_code_review_you_see_a_PR_mixing_a_log-parser_refactor.md>)
- [Intermediate] [Your ETL PR bundled a connector refactor, a new dbt model, and a partition-key fix in one commit, and reverting the bad partition change would also drop the model. Explain concretely why this hurts revertability during a data incident and how you'd coach the restructure.](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-28028_Your_ETL_PR_bundled_a_connector_refactor_a_new_dbt_model_and.md>)
- [Intermediate] [During code review you notice a PR mixes a refactor, a feature, and a bugfix in one commit. Why does this matter for revertability and bisect, and how would you coach the author to restructure it?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-28213_During_code_review_you_notice_a_PR_mixes_a_refactor_a_featur.md>)
- [Intermediate] [You commit on a detached HEAD during a bisect and the commits look orphaned afterward. Explain what happened at the ref level, how to save the work into a branch, and when gc would eventually reclaim them.](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-28396_You_commit_on_a_detached_HEAD_during_a_bisect_and_the_commit.md>)
- [Deep] [You pin a Hugging Face model by branch name (main) in your inference code and a silent model update broke output format overnight. Why is pinning by branch dangerous and how do you pin by immutable revision?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27365_You_pin_a_Hugging_Face_model_by_branch_name_main_in_your_inf.md>)
- [Deep] [A Spark job's config and its transformation code both changed in one PR, and a reviewer can't tell whether a data-quality drop came from the shuffle-partition change or the join logic. How would you split concerns across commits so a bad output is bisectable to config or code?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27983_A_Spark_job_s_config_and_its_transformation_code_both_change.md>)

## Related concepts

- [Resilience to Git Outages](<../../../10_DevOps_and_Platform/Delivery/GitOps/Reconciliation_Failures/Resilience_to_Git_Outages.md>) *(GitOps)*
- [Pinning Third-party Actions](<../GitHub_Actions/Pinning_Third-party_Actions.md>) *(GitHub Actions)*
- [Pull Request Pipelines](<../../../10_DevOps_and_Platform/Delivery/CI_CD/Pull_Request_Pipelines.md>) *(CI CD)*

