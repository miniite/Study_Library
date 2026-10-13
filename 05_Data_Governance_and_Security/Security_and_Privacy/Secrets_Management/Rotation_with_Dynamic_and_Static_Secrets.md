---
id: "05_Data_Governance_and_Security/Security_and_Privacy/Secrets_Management/Rotation_with_Dynamic_and_Static_Secrets"
level: 5
type: "concept"
topic: "05_Data_Governance_and_Security/Security_and_Privacy/Secrets_Management"
tools: ["HashiCorp_Vault", "AWS"]
---

# Rotation with Dynamic and Static Secrets

**L5 Expert** · [↑ Secrets Management](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [HashiCorp Vault](<../../../TOOLS/HashiCorp_Vault/_index.md>)
- [AWS](<../../../TOOLS/AWS/_index.md>)

## Practice questions

- [Intermediate] [Your app can reach Vault but gets TLS handshake errors after a cert rotation. How do you debug the trust chain?](<../../../PRACTICE/05_Data_Governance_and_Security/Security_and_Privacy/Secrets_Management/Q-39216_Your_app_can_reach_Vault_but_gets_TLS_handshake_errors_after.md>)
- [Deep] [Explain how the database secrets engine's connection root_rotation_statements protect the master DB credential.](<../../../PRACTICE/05_Data_Governance_and_Security/Security_and_Privacy/Secrets_Management/Q-39197_Explain_how_the_database_secrets_engine_s_connection_root_ro.md>)
- [Deep] [How do you architect Vault-issued SSH certificates to replace static keys and jump-host sprawl for a 2000-server fleet? Compare signed-cert vs OTP mode operationally and for audit.](<../../../PRACTICE/05_Data_Governance_and_Security/Security_and_Privacy/Secrets_Management/Q-39279_How_do_you_architect_Vault-issued_SSH_certificates_to_replac.md>)
- [Expert] [Your PKI engine issued a cert with a 90-day TTL but you need to revoke it now. Walk through revocation and CRL/OCSP implications.](<../../../PRACTICE/05_Data_Governance_and_Security/Security_and_Privacy/Secrets_Management/Q-39185_Your_PKI_engine_issued_a_cert_with_a_90-day_TTL_but_you_need.md>)
- [Expert] [What's the difference between Vault's SSH signed-certificate and OTP modes, and which scales better for a large fleet?](<../../../PRACTICE/05_Data_Governance_and_Security/Security_and_Privacy/Secrets_Management/Q-39219_What_s_the_difference_between_Vault_s_SSH_signed-certificate.md>)
- [Expert] [How do you handle secret rotation for a database that's read by both Vault dynamic creds and legacy static creds during migration?](<../../../PRACTICE/05_Data_Governance_and_Security/Security_and_Privacy/Secrets_Management/Q-39236_How_do_you_handle_secret_rotation_for_a_database_that_s_read.md>)
- [Expert] [How would you periodically rekey Vault (rotate the master encryption key and unseal shares) with minimal risk to a live cluster?](<../../../PRACTICE/05_Data_Governance_and_Security/Security_and_Privacy/Secrets_Management/Q-39243_How_would_you_periodically_rekey_Vault_rotate_the_master_enc.md>)
- [Expert] [Your identity provider is migrating from a legacy SAML setup to OIDC. Design the Vault identity/auth transition so 10000 human and machine identities keep working through the cutover with no downtime.](<../../../PRACTICE/05_Data_Governance_and_Security/Security_and_Privacy/Secrets_Management/Q-39276_Your_identity_provider_is_migrating_from_a_legacy_SAML_setup.md>)

## Related concepts

- [Azure Key Vault and Secrets](<../../../TOOLS/Azure/Azure_Platform/Azure_Key_Vault_and_Secrets.md>) *(Azure Platform)*
- [Federated Database Identity](<../../../01_Databases_and_SQL/Database_Operations/Security_and_Access/Column-level_Access/Federated_Database_Identity.md>) *(Security and Access)*
- [Secrets and Key Management](<../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Identity_and_Access_Foundations/Secrets_and_Key_Management.md>) *(Cloud Fundamentals)*

