---
id: "04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Proving_PII_Residency"
level: 5
type: "deep-dive"
topic: "04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture"
tools: ["AWS"]
---

# Proving PII Residency

**L5 Expert** · [↑ Data Architecture](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [AWS](<../../../../TOOLS/AWS/_index.md>)

## Practice questions

- [Intermediate] [You're asked to make onboarding a new data source self-serve for other teams. What do you make mandatory at registration (schema, owner, PII flag, SLA) and how do you block ungoverned side-door ingestion?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-11104_You_re_asked_to_make_onboarding_a_new_data_source_self-serve.md>)
- [Deep] [You need PII columns for a model but compliance says they can't sit unmasked in the lake. How do you get usable features while respecting masking/tokenization?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10671_You_need_PII_columns_for_a_model_but_compliance_says_they_ca.md>)
- [Deep] [You have PII columns in a silver table that analysts shouldn't see unmasked. How would you architect masking so it works whether they query via Spark or the warehouse, without maintaining two copies?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-11077_You_have_PII_columns_in_a_silver_table_that_analysts_shouldn.md>)
- [Deep] [Your mesh's federated governance council has become a bottleneck approving every new PII classification, so teams route around it. As principal, how do you architect policy-as-code with automated PII detection and default-deny so governance is enforced at write time without a human in the loop for the common case?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-11008_Your_mesh_s_federated_governance_council_has_become_a_bottle.md>)
- [Deep] [How do you architect the ingestion tier so onboarding a new source is self-serve config, not an engineering project, while still mandatorily attaching schema, PII classification, ownership, and SLA at registration time? What's the paved-road contract, and how do you block ungoverned side-door ingestion?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-11033_How_do_you_architect_the_ingestion_tier_so_onboarding_a_new_.md>)
- [Expert] [Design a multi-region data architecture for a global product with data-residency laws in the EU India and China. How do you keep a coherent analytical view while data physically can't leave certain regions?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10560_Design_a_multi-region_data_architecture_for_a_global_product.md>)
- [Expert] [How do you architect the ingestion tier to be self-serve so domains onboard new sources without the platform team, while keeping schema registration, PII classification, and cataloging automatic and mandatory?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10611_How_do_you_architect_the_ingestion_tier_to_be_self-serve_so_.md>)
- [Expert] [A regulator demands you prove no PII from EU customers ever entered a US-hosted dataset. Given a sprawling lake, how do you architect for provable data-flow containment rather than after-the-fact scanning?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10620_A_regulator_demands_you_prove_no_PII_from_EU_customers_ever_.md>)

## Related concepts

- [Fine-tuned Models as Sensitive Artifacts](<../../../../11_AI_and_ML/Responsible_AI/Privacy_and_PII/Fine-tuned_Models_as_Sensitive_Artifacts.md>) *(Privacy and PII)*
- [Access Governance for LLM Pipelines](<../../../../05_Data_Governance_and_Security/Security_and_Privacy/Access_Governance/Access_Governance_for_LLM_Pipelines.md>) *(Access Governance)*
- [Compliant Access to Encrypted PII](<../../../../05_Data_Governance_and_Security/Security_and_Privacy/Encryption_and_Compliance/Compliant_Access_to_Encrypted_PII.md>) *(Encryption and Compliance)*

