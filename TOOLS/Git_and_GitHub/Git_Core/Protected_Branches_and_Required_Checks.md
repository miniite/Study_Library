---
id: "TOOLS/Git_and_GitHub/Git_Core/Protected_Branches_and_Required_Checks"
level: 4
type: "concept"
topic: "TOOLS/Git_and_GitHub/Git_Core"
tools: ["Git_and_GitHub"]
---

# Protected Branches and Required Checks

**L4 Advanced** · [↑ Git Core](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Git and GitHub](<../_index.md>)

## Practice questions

- [Basic] [How would you configure Git so that git pull always rebases by default across your team's machines?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-28510_How_would_you_configure_Git_so_that_git_pull_always_rebases_.md>)
- [Intermediate] [What is a CODEOWNERS file and how would a DevOps team use it to require review of any change to the deployment folder?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-28491_What_is_a_CODEOWNERS_file_and_how_would_a_DevOps_team_use_it.md>)
- [Deep] [You want to enforce that any change to the migrations folder requires two approvals including a DBA, and that changes to reference-data seeds require only one. How would you express path-scoped, count-based review rules and prove they can't be bypassed?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27860_You_want_to_enforce_that_any_change_to_the_migrations_folder.md>)
- [Deep] [You need any change to the safety/guardrail prompt directory to require a review from the trust-and-safety team. How do you express that with CODEOWNERS path rules and what breaks if a glob is wrong?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27381_You_need_any_change_to_the_safety_guardrail_prompt_directory.md>)
- [Deep] [Your team standardized on git pull --rebase, but engineers occasionally rewrite already-pushed alert-rule commits, orphaning the deploy history other on-call folks rely on. What layered guardrails (config, hooks, branch protection, CI) keep rebase strictly local-only for the alerting repo?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27652_Your_team_standardized_on_git_pull_--rebase_but_engineers_oc.md>)
- [Deep] [You must guarantee a dependency's exact source, not just a semver, pinned to a Git commit for a supply-chain-sensitive service. Compare package-manager git refs, submodules, and vendoring for reproducibility and security.](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-28361_You_must_guarantee_a_dependency_s_exact_source_not_just_a_se.md>)
- [Deep] [You want blue-green deploys where Git holds both color manifests and a switch, but a partial merge left the switch pointing at a color with no running pods. How would you make the color switch atomic in Git and validate it before the controller acts?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-28592_You_want_blue-green_deploys_where_Git_holds_both_color_manif.md>)
- [Expert] [Your deploy job must only ever ship a commit that passed all required checks on main, never a re-run against a stale ref. Wire GitHub environments, deployment protection rules, and ref pinning to enforce that end to end.](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-28324_Your_deploy_job_must_only_ever_ship_a_commit_that_passed_all.md>)

## Related concepts

- [Resilience to Git Outages](<../../../10_DevOps_and_Platform/Delivery/GitOps/Reconciliation_Failures/Resilience_to_Git_Outages.md>) *(GitOps)*
- [Reusable Workflows](<../GitHub_Actions/Reusable_Workflows.md>) *(GitHub Actions)*
- [Pinning Third-party Actions](<../GitHub_Actions/Pinning_Third-party_Actions.md>) *(GitHub Actions)*

