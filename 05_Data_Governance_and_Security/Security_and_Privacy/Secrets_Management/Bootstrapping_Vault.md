---
id: "05_Data_Governance_and_Security/Security_and_Privacy/Secrets_Management/Bootstrapping_Vault"
level: 5
type: "concept"
topic: "05_Data_Governance_and_Security/Security_and_Privacy/Secrets_Management"
tools: ["HashiCorp_Vault", "AWS", "Terraform", "Kubernetes"]
---

# Bootstrapping Vault

**L5 Expert** · [↑ Secrets Management](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [HashiCorp Vault](<../../../TOOLS/HashiCorp_Vault/_index.md>)
- [AWS](<../../../TOOLS/AWS/_index.md>)
- [Terraform](<../../../TOOLS/Terraform/_index.md>)
- [Kubernetes](<../../../TOOLS/Kubernetes/_index.md>)

## Practice questions

- [Basic] [Why is it dangerous to bake a Vault token into a container image, and what pattern replaces it?](<../../../PRACTICE/05_Data_Governance_and_Security/Security_and_Privacy/Secrets_Management/Q-39156_Why_is_it_dangerous_to_bake_a_Vault_token_into_a_container_i.md>)
- [Intermediate] [Walk me through securing the Vault-to-storage and Vault-to-client communication with TLS in a k8s deployment.](<../../../PRACTICE/05_Data_Governance_and_Security/Security_and_Privacy/Secrets_Management/Q-39215_Walk_me_through_securing_the_Vault-to-storage_and_Vault-to-c.md>)
- [Deep] [Explain the difference between auto-unseal with a cloud KMS and manual Shamir unseal, and when you'd choose each in production.](<../../../PRACTICE/05_Data_Governance_and_Security/Security_and_Privacy/Secrets_Management/Q-39148_Explain_the_difference_between_auto-unseal_with_a_cloud_KMS_.md>)
- [Deep] [Why is putting Vault secrets into Terraform state a real risk even with a remote encrypted backend?](<../../../PRACTICE/05_Data_Governance_and_Security/Security_and_Privacy/Secrets_Management/Q-39164_Why_is_putting_Vault_secrets_into_Terraform_state_a_real_ris.md>)
- [Deep] [How do you inject secrets into a serverless function (Lambda/Cloud Run) that has cold starts, without Vault Agent sidecars?](<../../../PRACTICE/05_Data_Governance_and_Security/Security_and_Privacy/Secrets_Management/Q-39204_How_do_you_inject_secrets_into_a_serverless_function_Lambda_.md>)
- [Deep] [Explain how you'd wire Vault's transit engine to do envelope encryption for data-at-rest in an application database.](<../../../PRACTICE/05_Data_Governance_and_Security/Security_and_Privacy/Secrets_Management/Q-39241_Explain_how_you_d_wire_Vault_s_transit_engine_to_do_envelope.md>)
- [Deep] [How do you keep Terraform runs from leaking Vault-sourced credentials into state files across dozens of pipelines? Design the pattern and explain what still ends up in state regardless.](<../../../PRACTICE/05_Data_Governance_and_Security/Security_and_Privacy/Secrets_Management/Q-39305_How_do_you_keep_Terraform_runs_from_leaking_Vault-sourced_cr.md>)
- [Expert] [How do you decide between cloud-KMS auto-unseal and HSM-backed seal for a regulated workload, and what's the recovery and key-custody story you present to a security board for each?](<../../../PRACTICE/05_Data_Governance_and_Security/Security_and_Privacy/Secrets_Management/Q-39266_How_do_you_decide_between_cloud-KMS_auto-unseal_and_HSM-back.md>)

## Related concepts

- [Azure Key Vault and Secrets](<../../../TOOLS/Azure/Azure_Platform/Azure_Key_Vault_and_Secrets.md>) *(Azure Platform)*
- [Secrets and Key Management](<../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Identity_and_Access_Foundations/Secrets_and_Key_Management.md>) *(Cloud Fundamentals)*
- [Secrets in Pipelines](<../../../10_DevOps_and_Platform/Delivery/CI_CD/Secrets_in_Pipelines/_index.md>) *(CI CD)*

