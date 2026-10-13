---
id: "TOOLS/Git_and_GitHub/Git_Core/Server-side_Hooks_and_Push_Protection"
level: 4
type: "concept"
topic: "TOOLS/Git_and_GitHub/Git_Core"
tools: ["Git_and_GitHub", "AWS"]
---

# Server-side Hooks and Push Protection

**L4 Advanced** · [↑ Git Core](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Git and GitHub](<../_index.md>)
- [AWS](<../../AWS/_index.md>)

## Practice questions

- [Intermediate] [How would you write a simple bash pre-commit hook that blocks a commit if it contains the word 'password' or an AWS key pattern?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-28485_How_would_you_write_a_simple_bash_pre-commit_hook_that_block.md>)
- [Intermediate] [You want a pre-commit hook that blocks committing any file over 50 MB or a notebook with non-empty output cells, but a teammate committed via a client without the hook. Why can't client hooks be a real gate, and how do you enforce it server-side or in CI?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27196_You_want_a_pre-commit_hook_that_blocks_committing_any_file_o.md>)
- [Intermediate] [You want a pre-commit hook that blocks committing a notebook with non-empty output cells (which have leaked log samples before) or any file over 50 MB in the telemetry-analysis repo, but a teammate committed via a client without the hook. Why can't client hooks be a real gate, and how do you enforce it in CI?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27625_You_want_a_pre-commit_hook_that_blocks_committing_a_notebook.md>)
- [Intermediate] [Your Next.js app's build output leaks environment secrets into the bundle occasionally. How would you use gitleaks or push-protection plus pre-commit hooks to stop secrets reaching GitHub, and what's the gap client-side hooks leave?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-28216_Your_Next_js_app_s_build_output_leaks_environment_secrets_in.md>)
- [Deep] [You want to enforce that no migration file over a size threshold (a giant generated bulk-insert) or matching a connection-string pattern ever reaches the remote. Compare a server-side pre-receive hook, GitHub push protection, and a client-side hook, and the gap each leaves.](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27850_You_want_to_enforce_that_no_migration_file_over_a_size_thres.md>)
- [Deep] [You want to enforce that no committed prompt template contains a hardcoded PII example or a real customer transcript. Compare a server-side pre-receive hook, GitHub push protection, and a client hook for this policy.](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27436_You_want_to_enforce_that_no_committed_prompt_template_contai.md>)
- [Deep] [Your team enforces server-side push protection, but a determined dev could still commit a secret locally. Compare a pre-receive hook, GitHub push protection, and client-side hooks for blocking an AWS key, including the gap each leaves.](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-28331_Your_team_enforces_server-side_push_protection_but_a_determi.md>)
- [Expert] [You want a server-side pre-receive hook that rejects any push adding a migration file whose SQL matches a destructive pattern (DROP DATABASE, DELETE without WHERE) unless a bypass label is present. How do you implement it efficiently and what edge cases (force-push, deletes) break it?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27827_You_want_a_server-side_pre-receive_hook_that_rejects_any_pus.md>)

## Related concepts

- [Pinning Third-party Actions](<../GitHub_Actions/Pinning_Third-party_Actions.md>) *(GitHub Actions)*
- [Reusable Workflows](<../GitHub_Actions/Reusable_Workflows.md>) *(GitHub Actions)*
- [Resilience to Git Outages](<../../../10_DevOps_and_Platform/Delivery/GitOps/Reconciliation_Failures/Resilience_to_Git_Outages.md>) *(GitOps)*

