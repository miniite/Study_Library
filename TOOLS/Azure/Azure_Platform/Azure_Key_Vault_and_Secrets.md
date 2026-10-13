---
id: "TOOLS/Azure/Azure_Platform/Azure_Key_Vault_and_Secrets"
level: 4
type: "concept"
topic: "TOOLS/Azure/Azure_Platform"
tools: ["Azure", "HashiCorp_Vault", "Kubernetes"]
---

# Azure Key Vault and Secrets

**L4 Advanced** · [↑ Azure Platform](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Azure](<../_index.md>)
- [HashiCorp Vault](<../../HashiCorp_Vault/_index.md>)
- [Kubernetes](<../../Kubernetes/_index.md>)

## Practice questions

- [Basic] [What is Azure Active Directory (Microsoft Entra ID) and what is it used for?](<../../../PRACTICE/TOOLS/Azure/Azure_Platform/Q-03728_What_is_Azure_Active_Directory_Microsoft_Entra_ID_and_what_i.md>)
- [Intermediate] [Why is it better to reference secrets from Key Vault in a pipeline instead of hardcoding them in YAML?](<../../../PRACTICE/TOOLS/Azure/Azure_Platform/Q-03794_Why_is_it_better_to_reference_secrets_from_Key_Vault_in_a_pi.md>)
- [Intermediate] [What is a SAS token in Azure Storage and how is it different from a storage account key?](<../../../PRACTICE/TOOLS/Azure/Azure_Platform/Q-03983_What_is_a_SAS_token_in_Azure_Storage_and_how_is_it_different.md>)
- [Deep] [How would you make an Azure Storage account more secure (disable public access, use private endpoints, rotate keys)?](<../../../PRACTICE/TOOLS/Azure/Azure_Platform/Q-03985_How_would_you_make_an_Azure_Storage_account_more_secure_disa.md>)
- [Deep] [How would you enforce that no VM in prod has a public IP and every storage account uses private endpoints, using Azure Policy with deny and remediation?](<../../../PRACTICE/TOOLS/Azure/Azure_Platform/Q-03729_How_would_you_enforce_that_no_VM_in_prod_has_a_public_IP_and.md>)
- [Deep] [How do you handle secrets rotation end-to-end for an app on App Service pulling from Key Vault, so a rotated secret is picked up without a restart?](<../../../PRACTICE/TOOLS/Azure/Azure_Platform/Q-03759_How_do_you_handle_secrets_rotation_end-to-end_for_an_app_on_.md>)
- [Deep] [How would you set up automated compliance reporting (CIS/PCI) on Azure using Defender for Cloud and Policy, and remediate at scale?](<../../../PRACTICE/TOOLS/Azure/Azure_Platform/Q-03798_How_would_you_set_up_automated_compliance_reporting_CIS_PCI_.md>)
- [Expert] [Design an org-wide secret and certificate lifecycle strategy on Azure Key Vault with automated rotation across thousands of workloads; how do you handle the blast radius of a compromised vault?](<../../../PRACTICE/TOOLS/Azure/Azure_Platform/Q-03829_Design_an_org-wide_secret_and_certificate_lifecycle_strategy.md>)

## Related concepts

- [Secrets and Key Management](<../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Identity_and_Access_Foundations/Secrets_and_Key_Management.md>) *(Cloud Fundamentals)*
- [Vault Agent and Auto-auth](<../../../05_Data_Governance_and_Security/Security_and_Privacy/Secrets_Management/Vault_Agent_and_Auto-auth.md>) *(Secrets Management)*
- [Rotation with Dynamic and Static Secrets](<../../../05_Data_Governance_and_Security/Security_and_Privacy/Secrets_Management/Rotation_with_Dynamic_and_Static_Secrets.md>) *(Secrets Management)*

## References

- *Azure Data Fundamentals* — Michael John Pena — Data Roles and Responsibilities in Azure › Exam Tip
- *Azure Data Fundamentals* — Michael John Pena — Data Roles and Responsibilities in Azure › Beyond the Exam
- *Azure Data Fundamentals* — Michael John Pena — Azure Storage Solutions › Role-based access control: The digital gatekeeper
- *Azure Data Fundamentals* — Michael John Pena — Azure Storage Solutions › Shared access signatures: Temporary keys to your data kingdom
- *Microsoft Azure Storage: The Definitive Guide* — Avinash Valiramani — Storage account walkthrough › Download file from container
- *Microsoft Azure Storage: The Definitive Guide* — Avinash Valiramani — Azure Active Directory (Azure AD)
- *Microsoft Azure Storage: The Definitive Guide* — Avinash Valiramani — Shared Key
- *Microsoft Azure Storage: The Definitive Guide* — Avinash Valiramani — Shared access signature (SAS)
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 12: Azure SQL and Synapse Integration › Security Features
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 12: Azure SQL and Synapse Integration › Security in PolyBase and External Tables
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 12: Azure SQL and Synapse Integration › Security and Authentication
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 12: Azure SQL and Synapse Integration › Managed Identity Authentication
- *Data Governance with Unity Catalog on Databricks* — Kiran Sreekumar, Karthik Subbarao — Unity Catalog Under the Hood › Decoupled Storage Credentials
- *Data Governance with Unity Catalog on Databricks* — Kiran Sreekumar, Karthik Subbarao — Unity Catalog Under the Hood › Definition: Managed Identity
- *Data Governance with Unity Catalog on Databricks* — Kiran Sreekumar, Karthik Subbarao — Identity Management › Identity Provisioning
- *Data Governance with Unity Catalog on Databricks* — Kiran Sreekumar, Karthik Subbarao — M
- *Azure Data Engineering Cookbook* — Ahmad Osama — Working with Azure Blob Storage › How it works...
- *Azure Data Engineering Cookbook* — Ahmad Osama — Analyzing Data with Azure Synapse Analytics › How it works...
- *Azure Data Engineering Cookbook* — Ahmad Osama — Deploying Azure Data Factory Pipelines › Getting ready
- *Azure Data Factory Cookbook* — Xenia Ireton et al. — Working with Big Data -- HDInsight and Databricks › Preparing access credentials and code

