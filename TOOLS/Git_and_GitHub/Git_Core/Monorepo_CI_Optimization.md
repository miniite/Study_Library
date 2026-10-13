---
id: "TOOLS/Git_and_GitHub/Git_Core/Monorepo_CI_Optimization"
level: 4
type: "concept"
topic: "TOOLS/Git_and_GitHub/Git_Core"
tools: ["Git_and_GitHub", "Python"]
---

# Monorepo CI Optimization

**L4 Advanced** · [↑ Git Core](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Git and GitHub](<../_index.md>)
- [Python](<../../Python/_index.md>)

## Practice questions

- [Intermediate] [Why is it useful to write a small smoke test that a CI job runs to confirm a key model still builds and a critical row-count check still passes after a change?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27951_Why_is_it_useful_to_write_a_small_smoke_test_that_a_CI_job_r.md>)
- [Intermediate] [How would you make a CI pipeline only build the microservice whose files changed in a monorepo?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-28458_How_would_you_make_a_CI_pipeline_only_build_the_microservice.md>)
- [Intermediate] [A frontend PR needs different CI than a backend PR in the same monorepo. How do you structure workflows with path filters and reusable workflows so each app only runs its relevant checks?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-28264_A_frontend_PR_needs_different_CI_than_a_backend_PR_in_the_sa.md>)
- [Deep] [Why does a monorepo strategy affect how you branch and run CI compared to multiple repos?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-28085_Why_does_a_monorepo_strategy_affect_how_you_branch_and_run_C.md>)
- [Deep] [A path-filtered CI skipped retraining a model whose imported preprocessing library changed, because the model's own paths didn't. How do you make the change detection dependency-aware so it never skips a retrain it should have run?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27183_A_path-filtered_CI_skipped_retraining_a_model_whose_imported.md>)
- [Deep] [Argue monorepo versus polyrepo for a team owning a RAG service, an agent service, and an eval platform, specifically around atomic cross-service prompt changes, CI blast radius, and per-service access control.](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27435_Argue_monorepo_versus_polyrepo_for_a_team_owning_a_RAG_servi.md>)
- [Deep] [Your observability-config monorepo reruns promtool and dashboard-lint across every service on each PR, and CI now takes 20 minutes. Design path-filtered change detection so only the changed service's rules validate, including when a shared recording-rule library changes that a service imports.](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27585_Your_observability-config_monorepo_reruns_promtool_and_dashb.md>)
- [Deep] [A path-filtered CI skipped validating an Airflow DAG because only a shared operators library changed, not the DAG's own file, and a broken import shipped to the scheduler. How do you make change detection dependency-aware so it never skips a DAG it should have re-validated?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27989_A_path-filtered_CI_skipped_validating_an_Airflow_DAG_because.md>)

## Related concepts

- [Shared Pipeline Changes](<../../../10_DevOps_and_Platform/Delivery/CI_CD/Shared_Pipeline_Changes/_index.md>) *(CI CD)*
- [Deployment Gating Trade-offs](<../../../10_DevOps_and_Platform/Delivery/CI_CD/Shared_Pipeline_Changes/Deployment_Gating_Trade-offs.md>) *(CI CD)*
- [Idempotent Deploy Jobs](<../../../10_DevOps_and_Platform/Delivery/CI_CD/Idempotent_Deploy_Jobs/_index.md>) *(CI CD)*

