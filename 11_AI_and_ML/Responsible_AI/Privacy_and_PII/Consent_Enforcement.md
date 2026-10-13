---
id: "11_AI_and_ML/Responsible_AI/Privacy_and_PII/Consent_Enforcement"
level: 5
type: "concept"
topic: "11_AI_and_ML/Responsible_AI/Privacy_and_PII"
tools: ["Apache_Kafka"]
---

# Consent Enforcement

**L5 Expert** · [↑ Privacy and PII](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Apache Kafka](<../../../TOOLS/Apache_Kafka/_index.md>)

## Practice questions

- [Deep] [How would you architect PII scrubbing in a streaming pipeline (Kafka/Flink) where you can't buffer whole records but must mask before anything lands downstream?](<../../../PRACTICE/11_AI_and_ML/Responsible_AI/Privacy_and_PII/Q-14200_How_would_you_architect_PII_scrubbing_in_a_streaming_pipelin.md>)
- [Deep] [How do you architect consent enforcement for real-time streaming features so a consent revocation event stops a user's data flow within the same second across all consumers?](<../../../PRACTICE/11_AI_and_ML/Responsible_AI/Privacy_and_PII/Q-14298_How_do_you_architect_consent_enforcement_for_real-time_strea.md>)
- [Deep] [How do you architect a data-access-request approval flow with break-glass for PII that's both fast in a real incident and fully accountable after the fact?](<../../../PRACTICE/11_AI_and_ML/Responsible_AI/Privacy_and_PII/Q-14330_How_do_you_architect_a_data-access-request_approval_flow_wit.md>)
- [Expert] [How do you design consent propagation so that when a user revokes consent for marketing use, every feature and model that consumed that signal reflects it within SLA?](<../../../PRACTICE/11_AI_and_ML/Responsible_AI/Privacy_and_PII/Q-14156_How_do_you_design_consent_propagation_so_that_when_a_user_re.md>)
- [Expert] [Your feature store has a real-time and offline half. How do you guarantee consistent PII masking/consent enforcement across both without training-serving skew?](<../../../PRACTICE/11_AI_and_ML/Responsible_AI/Privacy_and_PII/Q-14184_Your_feature_store_has_a_real-time_and_offline_half_How_do_y.md>)

## Related concepts

- [Proving PII Residency](<../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Proving_PII_Residency.md>) *(Data Architecture)*
- [Compliant Access to Encrypted PII](<../../../05_Data_Governance_and_Security/Security_and_Privacy/Encryption_and_Compliance/Compliant_Access_to_Encrypted_PII.md>) *(Encryption and Compliance)*
- [Access Governance for LLM Pipelines](<../../../05_Data_Governance_and_Security/Security_and_Privacy/Access_Governance/Access_Governance_for_LLM_Pipelines.md>) *(Access Governance)*

