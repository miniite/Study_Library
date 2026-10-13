---
id: "05_Data_Governance_and_Security/Security_and_Privacy/Access_Governance/Governing_User-owned_Transformations"
level: 5
type: "concept"
topic: "05_Data_Governance_and_Security/Security_and_Privacy/Access_Governance"
tools: ["AWS"]
---

# Governing User-owned Transformations

**L5 Expert** · [↑ Access Governance](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [AWS](<../../../TOOLS/AWS/_index.md>)

## Practice questions

- [Deep] [Explain the security and governance implications of query-federation via Trino/Presto reaching into 15 source systems. How do you prevent the federation layer from becoming a privilege-escalation super-user?](<../../../PRACTICE/05_Data_Governance_and_Security/Security_and_Privacy/Access_Governance/Q-14657_Explain_the_security_and_governance_implications_of_query-fe.md>)
- [Deep] [How would you detect and prevent 'shadow data' — copies of governed datasets exported to notebooks S3 buckets and spreadsheets — from becoming an ungoverned exfiltration surface?](<../../../PRACTICE/05_Data_Governance_and_Security/Security_and_Privacy/Access_Governance/Q-14662_How_would_you_detect_and_prevent_shadow_data_copies_of_gover.md>)
- [Deep] [What's your framework for classifying an access-governance control as 'preventive detective or corrective' and why does a mature program need all three rather than over-investing in preventive gates?](<../../../PRACTICE/05_Data_Governance_and_Security/Security_and_Privacy/Access_Governance/Q-14697_What_s_your_framework_for_classifying_an_access-governance_c.md>)
- [Deep] [Why is 'grant access then review usage later' sometimes the right governance posture versus strict up-front denial, and how do you build the detective controls that make permissive-by-default safe?](<../../../PRACTICE/05_Data_Governance_and_Security/Security_and_Privacy/Access_Governance/Q-14709_Why_is_grant_access_then_review_usage_later_sometimes_the_ri.md>)
- [Deep] [An internal team wants to bypass governance 'just for a hackathon.' As the principal, how do you architect a safe fast-lane sandbox that satisfies them without creating a permanent governance hole?](<../../../PRACTICE/05_Data_Governance_and_Security/Security_and_Privacy/Access_Governance/Q-14719_An_internal_team_wants_to_bypass_governance_just_for_a_hacka.md>)
- [Deep] [How would you handle the governance gap between your data platform's access model and the underlying object storage's IAM — where a user denied at the SQL layer might still read raw files directly?](<../../../PRACTICE/05_Data_Governance_and_Security/Security_and_Privacy/Access_Governance/Q-14727_How_would_you_handle_the_governance_gap_between_your_data_pl.md>)
- [Expert] [What governance controls prevent a data-transformation job from silently escalating the sensitivity of an output (e.g. joining two low-risk tables to produce re-identifiable PII) and how do you enforce them in CI?](<../../../PRACTICE/05_Data_Governance_and_Security/Security_and_Privacy/Access_Governance/Q-14668_What_governance_controls_prevent_a_data-transformation_job_f.md>)
- [Expert] [How would you govern access to embeddings and vector stores derived from sensitive documents, given that embeddings can leak source content and bypass row/column controls entirely?](<../../../PRACTICE/05_Data_Governance_and_Security/Security_and_Privacy/Access_Governance/Q-14714_How_would_you_govern_access_to_embeddings_and_vector_stores_.md>)

## Related concepts

- [Database Access Control and Auditing](<../../../01_Databases_and_SQL/Database_Operations/Security_and_Access/Column-level_Access/Database_Access_Control_and_Auditing.md>) *(Security and Access)*
- [Policy as Code for Data](<../../Governance/Data_Strategy/Policy_as_Code_for_Data.md>) *(Data Strategy)*
- [Sensitive Data Controls for LLMs](<../../../11_AI_and_ML/Responsible_AI/AI_Governance/Sensitive_Data_Controls_for_LLMs.md>) *(AI Governance)*

## References

- *Data Governance with Unity Catalog on Databricks* — Kiran Sreekumar, Karthik Subbarao — Access Controls and Permissions Model

