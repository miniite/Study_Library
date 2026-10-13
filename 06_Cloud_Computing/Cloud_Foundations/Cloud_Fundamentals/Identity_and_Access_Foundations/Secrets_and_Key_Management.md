---
id: "06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Secrets_and_Key_Management"
level: 5
type: "deep-dive"
topic: "06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals"
tools: ["AWS", "Azure", "HashiCorp_Vault", "Kubernetes", "Google_Cloud", "Python"]
---

# Secrets and Key Management

**L5 Expert** · [↑ Cloud Fundamentals](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [AWS](<../../../../TOOLS/AWS/_index.md>)
- [Azure](<../../../../TOOLS/Azure/_index.md>)
- [HashiCorp Vault](<../../../../TOOLS/HashiCorp_Vault/_index.md>)
- [Kubernetes](<../../../../TOOLS/Kubernetes/_index.md>)
- [Google Cloud](<../../../../TOOLS/Google_Cloud/_index.md>)
- [Python](<../../../../TOOLS/Python/_index.md>)

## Practice questions

- [Basic] [How would you securely store your OpenAI or Anthropic API key when your app runs on a cloud instance?](<../../../../PRACTICE/06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Q-08364_How_would_you_securely_store_your_OpenAI_or_Anthropic_API_ke.md>)
- [Intermediate] [How would you store a database password so your app can read it but it never appears in your code repo?](<../../../../PRACTICE/06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Q-09008_How_would_you_store_a_database_password_so_your_app_can_read.md>)
- [Intermediate] [An interviewer asks why you would not store the database password in the container image even though the image is 'private'. What layers of exposure does a baked-in secret create?](<../../../../PRACTICE/06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Q-09184_An_interviewer_asks_why_you_would_not_store_the_database_pas.md>)
- [Deep] [An interviewer asks how you would prevent a single leaked API token from exfiltrating your entire customer database. What scoping, egress, and detection layers would you put in place?](<../../../../PRACTICE/06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Q-09166_An_interviewer_asks_how_you_would_prevent_a_single_leaked_AP.md>)
- [Expert] [A service using IMDSv1 was found vulnerable to SSRF credential theft. Explain the attack and how enforcing IMDSv2 hop limits mitigates it.](<../../../../PRACTICE/06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Q-08676_A_service_using_IMDSv1_was_found_vulnerable_to_SSRF_credenti.md>)
- [Expert] [How would you architect the org's defense-in-depth against cloud data exfiltration, layering egress allowlisting, IAM scoping, encryption, and detection, so that no single control failure exposes the customer datastore?](<../../../../PRACTICE/06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Q-08146_How_would_you_architect_the_org_s_defense-in-depth_against_c.md>)
- [Expert] [Explain how you'd architect key management so a compromised inference service in one tenant cannot decrypt another tenant's model weights or feature data, and so key rotation is provable without a fleet-wide model reload.](<../../../../PRACTICE/06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Q-08316_Explain_how_you_d_architect_key_management_so_a_compromised_.md>)
- [Expert] [How do you architect an org-wide answer to 'where does trust originate' in the cloud, so that human, service, and CI identities all chain back to one auditable root without long-lived keys anywhere?](<../../../../PRACTICE/06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Q-08881_How_do_you_architect_an_org-wide_answer_to_where_does_trust_.md>)

## Related concepts

- [Shared Secret Rotation](<../../../../05_Data_Governance_and_Security/Security_and_Privacy/Secrets_Management/Shared_Secret_Rotation.md>) *(Secrets Management)*
- [Encryption at Rest Strategy](<../../../../05_Data_Governance_and_Security/Security_and_Privacy/Encryption_and_Compliance/Encryption_at_Rest_Strategy.md>) *(Encryption and Compliance)*
- [Access Governance for LLM Pipelines](<../../../../05_Data_Governance_and_Security/Security_and_Privacy/Access_Governance/Access_Governance_for_LLM_Pipelines.md>) *(Access Governance)*

## References

- *Practical Data Privacy* — Katharine Jarmul — Building Privacy into Data Pipelines

