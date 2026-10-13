---
id: "TOOLS/Git_and_GitHub/Git_Core/Preventing_Secret_Leaks"
level: 4
type: "concept"
topic: "TOOLS/Git_and_GitHub/Git_Core"
tools: ["Git_and_GitHub", "AWS", "Python"]
---

# Preventing Secret Leaks

**L4 Advanced** · [↑ Git Core](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Git and GitHub](<../_index.md>)
- [AWS](<../../AWS/_index.md>)
- [Python](<../../Python/_index.md>)

## Practice questions

- [Basic] [What is a Git credential helper and how does it stop Git from asking for your password every push?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-28195_What_is_a_Git_credential_helper_and_how_does_it_stop_Git_fro.md>)
- [Basic] [You accidentally committed a .env file holding your Snowflake account and password and pushed it. What are the first steps you take?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27883_You_accidentally_committed_a_env_file_holding_your_Snowflake.md>)
- [Intermediate] [Why would you store your database connection string in a GitHub Actions secret rather than committing it into a workflow file?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27742_Why_would_you_store_your_database_connection_string_in_a_Git.md>)
- [Intermediate] [You accidentally committed a .env file containing your Anthropic key and pushed it. What are the first steps you take?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27261_You_accidentally_committed_a_env_file_containing_your_Anthro.md>)
- [Intermediate] [Why might you use gitleaks in your observability repo to catch an accidentally committed Grafana or PagerDuty API token?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27519_Why_might_you_use_gitleaks_in_your_observability_repo_to_cat.md>)
- [Deep] [A third-party GitHub Action in your migration pipeline was compromised and could exfiltrate your production database credentials. What supply-chain controls (SHA-pinning actions, least-privilege GITHUB_TOKEN, secret scoping) should have been in place, and your response now?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27838_A_third-party_GitHub_Action_in_your_migration_pipeline_was_c.md>)
- [Deep] [A leaked BigQuery service-account key was committed inside a training config, then removed in a follow-up commit. Explain why deleting it in a new commit is useless, and the correct order of rotate, purge history, and notify.](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27193_A_leaked_BigQuery_service-account_key_was_committed_inside_a.md>)
- [Deep] [You must guarantee no warehouse credential or cloud key ever reaches the remote from your pipeline repo. Compare gitleaks push-protection, a client-side pre-commit hook, and GitHub secret scanning, and the gap each leaves for a determined engineer.](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-28014_You_must_guarantee_no_warehouse_credential_or_cloud_key_ever.md>)

## Related concepts

- [Secrets in Actions](<../GitHub_Actions/Secrets_in_Actions.md>) *(GitHub Actions)*
- [Pinning Third-party Actions](<../GitHub_Actions/Pinning_Third-party_Actions.md>) *(GitHub Actions)*
- [Secrets in GitOps](<../../../10_DevOps_and_Platform/Delivery/GitOps/Secrets_in_GitOps.md>) *(GitOps)*

