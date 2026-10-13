---
id: "10_DevOps_and_Platform/Delivery/CI_CD/Secrets_in_Pipelines"
level: 4
type: "concept"
topic: "10_DevOps_and_Platform/Delivery/CI_CD"
tools: ["AWS", "HashiCorp_Vault", "Jenkins", "Git_and_GitHub"]
---

# Secrets in Pipelines

**L4 Advanced** · [↑ CI CD](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Go deeper

- [Supply Chain Security and Provenance](<Supply_Chain_Security_and_Provenance.md>)

## Implementations (tools)

- [AWS](<../../../../TOOLS/AWS/_index.md>)
- [HashiCorp Vault](<../../../../TOOLS/HashiCorp_Vault/_index.md>)
- [Jenkins](<../../../../TOOLS/Jenkins/_index.md>)
- [Git and GitHub](<../../../../TOOLS/Git_and_GitHub/_index.md>)

## Practice questions

- [Basic] [What is the risk of hard-coding a Datadog API key directly inside a pipeline YAML file in the repo?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06458_What_is_the_risk_of_hard-coding_a_Datadog_API_key_directly_i.md>)
- [Basic] [How would you store a database password that the pipeline needs, safely?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07333_How_would_you_store_a_database_password_that_the_pipeline_ne.md>)
- [Intermediate] [How do you store secrets like API keys and passwords in a CI/CD pipeline?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06838_How_do_you_store_secrets_like_API_keys_and_passwords_in_a_CI.md>)
- [Deep] [How would you pin third-party pipeline actions/plugins to a specific SHA and why?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06776_How_would_you_pin_third-party_pipeline_actions_plugins_to_a_.md>)
- [Deep] [Your remediation automation and the pipeline both hold kube credentials; how would you enforce least privilege so the pipeline can deploy the runbook but only the runtime can execute cluster actions?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06629_Your_remediation_automation_and_the_pipeline_both_hold_kube_.md>)
- [Deep] [Why is loading a pickled model from an untrusted source a security risk, and how would you gate against it in the pipeline?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07673_Why_is_loading_a_pickled_model_from_an_untrusted_source_a_se.md>)
- [Deep] [Your service reads secrets at startup, so rotating a secret requires a redeploy and rotations became rare; how would you change the pipeline and app so secret rotation is decoupled from deploys?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06980_Your_service_reads_secrets_at_startup_so_rotating_a_secret_r.md>)
- [Expert] [You need OIDC-based keyless auth from your pipeline to a cloud provider; what's the trust chain, what could a fork PR abuse, and how do you scope the role to prevent lateral movement?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06840_You_need_OIDC-based_keyless_auth_from_your_pipeline_to_a_clo.md>)

## Related concepts

- [Preventing Secret Leaks](<../../../../TOOLS/Git_and_GitHub/Git_Core/Preventing_Secret_Leaks.md>) *(Git Core)*
- [Secrets and Key Management](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Identity_and_Access_Foundations/Secrets_and_Key_Management.md>) *(Cloud Fundamentals)*
- [Securing ML Delivery](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Safe_Config_Rollouts/Securing_ML_Delivery.md>) *(Cloud Fundamentals)*

