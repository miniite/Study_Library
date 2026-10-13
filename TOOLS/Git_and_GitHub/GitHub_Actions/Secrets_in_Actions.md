---
id: "TOOLS/Git_and_GitHub/GitHub_Actions/Secrets_in_Actions"
level: 4
type: "concept"
topic: "TOOLS/Git_and_GitHub/GitHub_Actions"
tools: ["Git_and_GitHub", "AWS", "HashiCorp_Vault"]
---

# Secrets in Actions

**L4 Advanced** · [↑ GitHub Actions](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Git and GitHub](<../_index.md>)
- [AWS](<../../AWS/_index.md>)
- [HashiCorp Vault](<../../HashiCorp_Vault/_index.md>)

## Practice questions

- [Basic] [What are GitHub Actions secrets and why should you use them instead of hardcoding values?](<../../../PRACTICE/TOOLS/Git_and_GitHub/GitHub_Actions/Q-28702_What_are_GitHub_Actions_secrets_and_why_should_you_use_them_.md>)
- [Intermediate] [What is the difference between the env context and the secrets context?](<../../../PRACTICE/TOOLS/Git_and_GitHub/GitHub_Actions/Q-28705_What_is_the_difference_between_the_env_context_and_the_secre.md>)
- [Intermediate] [What happens if a secret is empty or missing when a step tries to use it, and how would you catch that?](<../../../PRACTICE/TOOLS/Git_and_GitHub/GitHub_Actions/Q-28758_What_happens_if_a_secret_is_empty_or_missing_when_a_step_tri.md>)
- [Deep] [Why does GitHub mask secrets in logs and can you still accidentally leak one?](<../../../PRACTICE/TOOLS/Git_and_GitHub/GitHub_Actions/Q-28759_Why_does_GitHub_mask_secrets_in_logs_and_can_you_still_accid.md>)
- [Deep] [A secret leaked into logs despite masking. How does masking actually work, why did it fail, and how do you prevent recurrence?](<../../../PRACTICE/TOOLS/Git_and_GitHub/GitHub_Actions/Q-28804_A_secret_leaked_into_logs_despite_masking_How_does_masking_a.md>)
- [Deep] [You migrated from Jenkins to GitHub Actions. What did you have to redesign around shared libraries, credentials, and agents?](<../../../PRACTICE/TOOLS/Git_and_GitHub/GitHub_Actions/Q-28854_You_migrated_from_Jenkins_to_GitHub_Actions_What_did_you_hav.md>)
- [Deep] [When a workflow needs a value from Parameter Store or an SSM secret at runtime, how do you fetch it without persisting it in the environment?](<../../../PRACTICE/TOOLS/Git_and_GitHub/GitHub_Actions/Q-28878_When_a_workflow_needs_a_value_from_Parameter_Store_or_an_SSM.md>)
- [Expert] [How would you integrate a secrets manager like Vault or AWS Secrets Manager so short-lived secrets are fetched at runtime instead of stored as repo secrets?](<../../../PRACTICE/TOOLS/Git_and_GitHub/GitHub_Actions/Q-28805_How_would_you_integrate_a_secrets_manager_like_Vault_or_AWS_.md>)

## Related concepts

- [Preventing Secret Leaks](<../Git_Core/Preventing_Secret_Leaks.md>) *(Git Core)*
- [Secrets in GitOps](<../../../10_DevOps_and_Platform/Delivery/GitOps/Secrets_in_GitOps.md>) *(GitOps)*
- [Secure Workflows for Forks](<../Git_Core/Secure_Workflows_for_Forks.md>) *(Git Core)*

## References

- *Learning GitHub Actions* — Brent Laster — Actions and Security

