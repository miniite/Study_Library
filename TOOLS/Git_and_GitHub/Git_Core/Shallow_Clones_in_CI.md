---
id: "TOOLS/Git_and_GitHub/Git_Core/Shallow_Clones_in_CI"
level: 4
type: "concept"
topic: "TOOLS/Git_and_GitHub/Git_Core"
tools: ["Git_and_GitHub"]
---

# Shallow Clones in CI

**L4 Advanced** · [↑ Git Core](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Git and GitHub](<../_index.md>)

## Practice questions

- [Intermediate] [Your migration CI runs against a shallow clone and git describe fails, so the schema-version string baked into the release comes out wrong. Explain why depth-limited clones lose tag reachability and how you'd fix versioning without cloning full history.](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27816_Your_migration_CI_runs_against_a_shallow_clone_and_git_descr.md>)
- [Intermediate] [In a CI pipeline, why might a shallow clone break git describe or blame-based tooling, and how do you fix it?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-28187_In_a_CI_pipeline_why_might_a_shallow_clone_break_git_describ.md>)
- [Intermediate] [A training job in CI uses a shallow clone for speed, and git describe fails so your model version string comes out wrong. Explain why depth-limited clones lose tag reachability and how you'd fix versioning without cloning full history.](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27187_A_training_job_in_CI_uses_a_shallow_clone_for_speed_and_git_.md>)
- [Intermediate] [A telemetry-detector training job in CI passes locally but fails on the runner with 'dubious ownership' and missing-tag errors. Explain how CI's root-owned, shallow, detached-HEAD checkout causes each error and the specific fix for each in an ML-on-telemetry pipeline.](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27662_A_telemetry-detector_training_job_in_CI_passes_locally_but_f.md>)
- [Deep] [What is a shallow clone and when would you use --depth in a CI pipeline?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-28653_What_is_a_shallow_clone_and_when_would_you_use_--depth_in_a_.md>)
- [Deep] [A fine-tune training job in CI passes locally but fails with 'dubious ownership' and missing-tag errors on the runner. Explain how CI's root-owned, shallow, detached-HEAD checkout causes each error and the specific fix.](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27446_A_fine-tune_training_job_in_CI_passes_locally_but_fails_with.md>)
- [Deep] [Your CI shallow-clones a huge data-platform monorepo for speed, but the Spark-job release step's git describe version string and blame tooling break. Explain why depth-limited clones lose tag reachability and how you'd recover versioning without cloning full history.](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-28001_Your_CI_shallow-clones_a_huge_data-platform_monorepo_for_spe.md>)
- [Deep] [A CI job using a shallow clone breaks git describe and blame-based coverage tooling. Explain why depth-limited clones lose that info and how you'd fix it without giving up all the speed benefit.](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-28246_A_CI_job_using_a_shallow_clone_breaks_git_describe_and_blame.md>)

## Related concepts

- [Resilience to Git Outages](<../../../10_DevOps_and_Platform/Delivery/GitOps/Reconciliation_Failures/Resilience_to_Git_Outages.md>) *(GitOps)*
- [Pinning Third-party Actions](<../GitHub_Actions/Pinning_Third-party_Actions.md>) *(GitHub Actions)*
- [Pre-merge vs Post-merge Failures](<../../../10_DevOps_and_Platform/Delivery/CI_CD/Pre-merge_vs_Post-merge_Failures.md>) *(CI CD)*

