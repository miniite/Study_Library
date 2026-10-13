---
id: "TOOLS/Git_and_GitHub/Git_Core/Sparse_Checkout_and_Monorepos"
level: 4
type: "concept"
topic: "TOOLS/Git_and_GitHub/Git_Core"
tools: ["Git_and_GitHub"]
---

# Sparse Checkout and Monorepos

**L4 Advanced** · [↑ Git Core](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Git and GitHub](<../_index.md>)

## Practice questions

- [Deep] [You want new database engineers to check out only their service's migrations subtree in a large monorepo. Explain cone-mode sparse-checkout, how it composes with a partial clone, and the gotchas when a migration references a shared seed file outside the cone.](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27876_You_want_new_database_engineers_to_check_out_only_their_serv.md>)
- [Deep] [Developer clones of your 8 GB GenAI monorepo (prompts, adapters, eval data) take 15 minutes. Compare partial clone with --filter=blob:none, shallow clone, and sparse-checkout, including what fetches lazily.](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27402_Developer_clones_of_your_8_GB_GenAI_monorepo_prompts_adapter.md>)
- [Deep] [You want new AI engineers to check out only the retrieval-service subtree of a large GenAI monorepo. Explain cone-mode sparse-checkout, how it composes with partial clone, and the gotchas when a build references files outside the cone.](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27403_You_want_new_AI_engineers_to_check_out_only_the_retrieval-se.md>)
- [Deep] [Developer clones of your 8 GB AIOps monorepo (dashboards, rules, runbooks, telemetry fixtures) take 15 minutes and thrash CI. Compare partial clone with --filter=blob:none, shallow clone, and sparse-checkout for this, including which operations fetch lazily and the offline failure modes during an incident.](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27639_Developer_clones_of_your_8_GB_AIOps_monorepo_dashboards_rule.md>)
- [Deep] [You want new AIOps engineers to check out only the alert-correlation service subtree of a large monorepo. Explain cone-mode sparse-checkout, how it composes with partial clone, and the gotcha when the runbook build references shared rule files outside the cone.](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27640_You_want_new_AIOps_engineers_to_check_out_only_the_alert-cor.md>)
- [Deep] [You want new engineers to check out only the ingestion service subtree of a large data monorepo, but their dbt build references shared macros outside the cone. Explain cone-mode sparse-checkout, how it composes with partial clone, and how to handle the out-of-cone dependency.](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-28003_You_want_new_engineers_to_check_out_only_the_ingestion_servi.md>)
- [Deep] [You want new engineers to clone only the part of the monorepo they work on. Explain sparse-checkout (cone mode) setup, how it interacts with partial clone, and the gotchas when files outside the cone are referenced.](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-28280_You_want_new_engineers_to_clone_only_the_part_of_the_monorep.md>)
- [Deep] [You want new backend engineers to check out only their service subtree in a large monorepo. Explain cone-mode sparse-checkout, how it composes with partial clone, and the gotchas when a build references files outside the cone.](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-28353_You_want_new_backend_engineers_to_check_out_only_their_servi.md>)

## Related concepts

- [Shared Pipeline Changes](<../../../10_DevOps_and_Platform/Delivery/CI_CD/Shared_Pipeline_Changes/_index.md>) *(CI CD)*
- [Ephemeral Environments](<../../../10_DevOps_and_Platform/Delivery/CI_CD/Ephemeral_Environments.md>) *(CI CD)*
- [Resilience to Git Outages](<../../../10_DevOps_and_Platform/Delivery/GitOps/Reconciliation_Failures/Resilience_to_Git_Outages.md>) *(GitOps)*

