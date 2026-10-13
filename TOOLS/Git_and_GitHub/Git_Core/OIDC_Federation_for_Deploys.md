---
id: "TOOLS/Git_and_GitHub/Git_Core/OIDC_Federation_for_Deploys"
level: 4
type: "concept"
topic: "TOOLS/Git_and_GitHub/Git_Core"
tools: ["AWS", "Git_and_GitHub"]
---

# OIDC Federation for Deploys

**L4 Advanced** · [↑ Git Core](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [AWS](<../../AWS/_index.md>)
- [Git and GitHub](<../_index.md>)

## Practice questions

- [Intermediate] [Explain how you'd give an external contractor push access to exactly one repo and one branch without granting broad org access, comparing deploy keys, fine-grained PATs, and outside-collaborator roles.](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-28276_Explain_how_you_d_give_an_external_contractor_push_access_to.md>)
- [Deep] [Your GitHub Actions job that applies migrations to a staging RDS uses long-lived AWS keys. Walk through migrating to OIDC federation and what ref/branch conditions in the trust policy stop a fork PR from assuming the deploy role.](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27839_Your_GitHub_Actions_job_that_applies_migrations_to_a_staging.md>)
- [Deep] [How would you let a GitHub Actions job authenticate to AWS without storing long-lived access keys as secrets?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-28469_How_would_you_let_a_GitHub_Actions_job_authenticate_to_AWS_w.md>)
- [Deep] [Your GitHub Actions deploy of a model to a SageMaker endpoint uses long-lived AWS keys. Walk through migrating to OIDC federation, and what ref/branch conditions in the trust policy prevent a fork PR from assuming the deploy role.](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27190_Your_GitHub_Actions_deploy_of_a_model_to_a_SageMaker_endpoin.md>)
- [Deep] [Your GitHub Actions deploy of a fine-tuned model to AWS uses long-lived keys. Walk through migrating to OIDC federation and what ref/branch conditions in the trust policy stop a fork PR from assuming the role.](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27415_Your_GitHub_Actions_deploy_of_a_fine-tuned_model_to_AWS_uses.md>)
- [Deep] [Give an external annotator team push access to exactly one eval-dataset repo and one branch without granting broad org access to your models code. Compare deploy keys, fine-grained PATs, and outside-collaborator roles.](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27442_Give_an_external_annotator_team_push_access_to_exactly_one_e.md>)
- [Deep] [Your GitHub Actions job that deploys updated alerting rules to your cloud monitoring backend uses long-lived API keys. Walk through migrating to OIDC federation, and what ref conditions in the trust policy prevent a fork PR from assuming the deploy role and pushing rogue rules.](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27619_Your_GitHub_Actions_job_that_deploys_updated_alerting_rules_.md>)
- [Expert] [Why is OIDC-based authentication preferred over long-lived cloud keys in a GitHub Actions deploy job?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-28169_Why_is_OIDC-based_authentication_preferred_over_long-lived_c.md>)

## Related concepts

- [Secrets in Actions](<../GitHub_Actions/Secrets_in_Actions.md>) *(GitHub Actions)*
- [Pinning Third-party Actions](<../GitHub_Actions/Pinning_Third-party_Actions.md>) *(GitHub Actions)*
- [Reusable Workflows](<../GitHub_Actions/Reusable_Workflows.md>) *(GitHub Actions)*

