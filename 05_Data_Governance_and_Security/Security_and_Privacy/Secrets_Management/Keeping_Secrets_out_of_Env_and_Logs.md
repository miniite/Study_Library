---
id: "05_Data_Governance_and_Security/Security_and_Privacy/Secrets_Management/Keeping_Secrets_out_of_Env_and_Logs"
level: 5
type: "concept"
topic: "05_Data_Governance_and_Security/Security_and_Privacy/Secrets_Management"
tools: ["HashiCorp_Vault", "AWS"]
---

# Keeping Secrets out of Env and Logs

**L5 Expert** · [↑ Secrets Management](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [HashiCorp Vault](<../../../TOOLS/HashiCorp_Vault/_index.md>)
- [AWS](<../../../TOOLS/AWS/_index.md>)

## Practice questions

- [Basic] [Why do we prefer Vault's dynamic database secrets over storing a long-lived static DB password in a Kubernetes Secret?](<../../../PRACTICE/05_Data_Governance_and_Security/Security_and_Privacy/Secrets_Management/Q-39146_Why_do_we_prefer_Vault_s_dynamic_database_secrets_over_stori.md>)
- [Intermediate] [How do you handle secrets for ephemeral preview environments that spin up per pull request?](<../../../PRACTICE/05_Data_Governance_and_Security/Security_and_Privacy/Secrets_Management/Q-39199_How_do_you_handle_secrets_for_ephemeral_preview_environments.md>)
- [Deep] [What is response wrapping in Vault and how does it protect a SecretID in transit through your CI system?](<../../../PRACTICE/05_Data_Governance_and_Security/Security_and_Privacy/Secrets_Management/Q-39173_What_is_response_wrapping_in_Vault_and_how_does_it_protect_a.md>)
- [Deep] [Describe a rollout plan to introduce Vault into an org that currently keeps secrets in .env files and CI variables.](<../../../PRACTICE/05_Data_Governance_and_Security/Security_and_Privacy/Secrets_Management/Q-39221_Describe_a_rollout_plan_to_introduce_Vault_into_an_org_that_.md>)
- [Deep] [Explain how you'd wire dynamic database secrets for a fleet of 80 databases (Postgres, MySQL, MongoDB, Cassandra) with different connection lifecycles, and how you keep connection-pool churn from hammering the DBs.](<../../../PRACTICE/05_Data_Governance_and_Security/Security_and_Privacy/Secrets_Management/Q-39285_Explain_how_you_d_wire_dynamic_database_secrets_for_a_fleet_.md>)
- [Expert] [Your Vault stopped serving requests and the audit log filesystem was full. Why did Vault block, and how do you prevent this?](<../../../PRACTICE/05_Data_Governance_and_Security/Security_and_Privacy/Secrets_Management/Q-39169_Your_Vault_stopped_serving_requests_and_the_audit_log_filesy.md>)
- [Expert] [Present a build-vs-buy analysis for self-hosted Vault OSS versus HCP Vault Dedicated versus AWS Secrets Manager as your org-wide platform. What TCO and lock-in factors drive the recommendation?](<../../../PRACTICE/05_Data_Governance_and_Security/Security_and_Privacy/Secrets_Management/Q-39245_Present_a_build-vs-buy_analysis_for_self-hosted_Vault_OSS_ve.md>)
- [Expert] [A director challenges the ROI of Vault Enterprise versus OSS plus glue code. Build the case: which Enterprise features are non-negotiable at your scale, and which could you replicate with tooling?](<../../../PRACTICE/05_Data_Governance_and_Security/Security_and_Privacy/Secrets_Management/Q-39274_A_director_challenges_the_ROI_of_Vault_Enterprise_versus_OSS.md>)

## Related concepts

- [Secrets and Key Management](<../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Identity_and_Access_Foundations/Secrets_and_Key_Management.md>) *(Cloud Fundamentals)*
- [Azure Key Vault and Secrets](<../../../TOOLS/Azure/Azure_Platform/Azure_Key_Vault_and_Secrets.md>) *(Azure Platform)*
- [Secrets in Pipelines](<../../../10_DevOps_and_Platform/Delivery/CI_CD/Secrets_in_Pipelines/_index.md>) *(CI CD)*

