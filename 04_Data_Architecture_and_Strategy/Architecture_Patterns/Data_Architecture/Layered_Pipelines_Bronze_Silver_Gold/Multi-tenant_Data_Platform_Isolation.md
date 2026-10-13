---
id: "04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Multi-tenant_Data_Platform_Isolation"
level: 5
type: "deep-dive"
topic: "04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture"
tools: ["Apache_Spark", "AWS", "Snowflake"]
---

# Multi-tenant Data Platform Isolation

**L5 Expert** · [↑ Data Architecture](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Apache Spark](<../../../../TOOLS/Apache_Spark/_index.md>)
- [AWS](<../../../../TOOLS/AWS/_index.md>)
- [Snowflake](<../../../../TOOLS/Snowflake/_index.md>)

## Practice questions

- [Deep] [Your team wants row-level and column-level access control on feature tables consumed across Spark and Snowflake. Why is enforcing this consistently hard?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10712_Your_team_wants_row-level_and_column-level_access_control_on.md>)
- [Deep] [How do you architect multi-tenant isolation in a shared data platform where a noisy tenant's query can starve others? Compare compute isolation, quotas, and separate accounts as strategies for a SaaS at scale.](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10590_How_do_you_architect_multi-tenant_isolation_in_a_shared_data.md>)
- [Deep] [Your data platform must support both interactive sub-second dashboards and hour-long batch jobs on the same physical cluster, and they contend for I/O. As principal, how do you architect physical and logical isolation (separate compute pools, storage tiering, query admission control) so neither workload starves the other under peak load?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-11039_Your_data_platform_must_support_both_interactive_sub-second_.md>)
- [Expert] [How do you architect encryption and key management across a lake so that per-tenant/per-region key isolation and crypto-shredding are possible without wrecking query performance or operational complexity?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10595_How_do_you_architect_encryption_and_key_management_across_a_.md>)
- [Expert] [Design a governance model that lets RAG and fine-tuning pipelines consume enterprise data from the lake while provably preventing PII and confidential fields from entering embeddings or prompts. What new controls does the data platform need?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10757_Design_a_governance_model_that_lets_RAG_and_fine-tuning_pipe.md>)
- [Expert] [Design the access-control model for feature tables consumed across Spark, Snowflake, and an online store by 5000 data scientists. Why is enforcing consistent row/column/masking policy across engines the hard part, and how do you avoid N policy copies?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10784_Design_the_access-control_model_for_feature_tables_consumed_.md>)
- [Expert] [Design the access-control model for a telemetry lake serving thousands of engineers where logs contain sensitive prod data. How do you unify row/column/masking policy across the query engines (Trino, the vendor UI, notebooks) without maintaining N copies of policy?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10880_Design_the_access-control_model_for_a_telemetry_lake_serving.md>)
- [Expert] [How do you architect encryption and key management across a telemetry lake so per-tenant/per-region key isolation and crypto-shredding of leaked-PII logs are possible without wrecking the columnar scan performance your AIOps queries depend on?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10902_How_do_you_architect_encryption_and_key_management_across_a_.md>)

## Related concepts

- [Database Access Control and Auditing](<../../../../01_Databases_and_SQL/Database_Operations/Security_and_Access/Column-level_Access/Database_Access_Control_and_Auditing.md>) *(Security and Access)*
- [Audit and Compliance Platforms](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Audit_and_Compliance_Platforms.md>) *(High Level Design)*
- [Cross-border Access Governance](<../../../../05_Data_Governance_and_Security/Security_and_Privacy/Access_Governance/Cross-border_Access_Governance.md>) *(Access Governance)*

## References

- *Data Governance with Unity Catalog on Databricks* — Kiran Sreekumar, Karthik Subbarao — Unity Catalog and Compute

