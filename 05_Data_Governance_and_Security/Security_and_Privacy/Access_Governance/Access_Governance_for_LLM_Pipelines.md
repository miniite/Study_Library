---
id: "05_Data_Governance_and_Security/Security_and_Privacy/Access_Governance/Access_Governance_for_LLM_Pipelines"
level: 5
type: "concept"
topic: "05_Data_Governance_and_Security/Security_and_Privacy/Access_Governance"
tools: ["AWS", "Azure"]
---

# Access Governance for LLM Pipelines

**L5 Expert** · [↑ Access Governance](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [AWS](<../../../TOOLS/AWS/_index.md>)
- [Azure](<../../../TOOLS/Azure/_index.md>)

## Practice questions

- [Deep] [Explain how you'd design a data-classification taxonomy that machines can enforce automatically at ingestion time versus one that only humans can apply. Where does auto-classification fail and how do you contain that risk?](<../../../PRACTICE/05_Data_Governance_and_Security/Security_and_Privacy/Access_Governance/Q-14645_Explain_how_you_d_design_a_data-classification_taxonomy_that.md>)
- [Deep] [An engineer says 'we'll just grant broad read access to the whole gold layer and trust people.' Push back architecturally — what does least-privilege at the semantic layer actually look like and what's the cost of getting it wrong?](<../../../PRACTICE/05_Data_Governance_and_Security/Security_and_Privacy/Access_Governance/Q-14670_An_engineer_says_we_ll_just_grant_broad_read_access_to_the_w.md>)
- [Deep] [An exec wants 'AI-powered auto-grant' where an LLM approves access requests. Assess the governance risk and design guardrails so the model can recommend but never unilaterally grant sensitive access.](<../../../PRACTICE/05_Data_Governance_and_Security/Security_and_Privacy/Access_Governance/Q-14693_An_exec_wants_AI-powered_auto-grant_where_an_LLM_approves_ac.md>)
- [Deep] [Explain how you'd prevent 'access accretion' where employees changing roles keep old data grants forever. Design the joiner-mover-leaver automation and how it ties to the data platform.](<../../../PRACTICE/05_Data_Governance_and_Security/Security_and_Privacy/Access_Governance/Q-14713_Explain_how_you_d_prevent_access_accretion_where_employees_c.md>)
- [Deep] [What's the risk of using a single 'god' pipeline service-account that all ETL runs under, and how would you re-architect toward per-job scoped identities without a massive operational overhead?](<../../../PRACTICE/05_Data_Governance_and_Security/Security_and_Privacy/Access_Governance/Q-14739_What_s_the_risk_of_using_a_single_god_pipeline_service-accou.md>)
- [Expert] [How do you govern access to derived and aggregated datasets so that a user denied the raw PII can't reconstruct it from three permitted aggregates? Walk me through your inference-control strategy.](<../../../PRACTICE/05_Data_Governance_and_Security/Security_and_Privacy/Access_Governance/Q-14648_How_do_you_govern_access_to_derived_and_aggregated_datasets_.md>)
- [Expert] [How do you architect purpose-based access control so the same analyst can query a dataset for fraud-detection but is blocked from using it for marketing, and how is 'purpose' enforced technically rather than by policy PDF?](<../../../PRACTICE/05_Data_Governance_and_Security/Security_and_Privacy/Access_Governance/Q-14671_How_do_you_architect_purpose-based_access_control_so_the_sam.md>)
- [Expert] [Describe your approach to separation-of-duties in a data platform so that no single person can both grant themselves access and delete the audit record of doing so — down to the IAM and pipeline level.](<../../../PRACTICE/05_Data_Governance_and_Security/Security_and_Privacy/Access_Governance/Q-14690_Describe_your_approach_to_separation-of-duties_in_a_data_pla.md>)

## Related concepts

- [Database Access Control and Auditing](<../../../01_Databases_and_SQL/Database_Operations/Security_and_Access/Column-level_Access/Database_Access_Control_and_Auditing.md>) *(Security and Access)*
- [Preventing PII Leakage](<../../Governance/Stewardship/Preventing_PII_Leakage.md>) *(Stewardship)*

## References

- *Cracking the Data Engineering Interview* — Kedeisha Bryan, Taamir Ransome — Data Security and Privacy
- *Cracking the Data Engineering Interview* — Kedeisha Bryan, Taamir Ransome — Understanding data access control
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Security Design Patterns

