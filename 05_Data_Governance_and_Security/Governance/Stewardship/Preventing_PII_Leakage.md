---
id: "05_Data_Governance_and_Security/Governance/Stewardship/Preventing_PII_Leakage"
level: 5
type: "concept"
topic: "05_Data_Governance_and_Security/Governance/Stewardship"
tools: []
---

# Preventing PII Leakage

**L5 Expert** · [↑ Stewardship](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Practice questions

- [Deep] [What governance controls would you put in place so that PII does not leak from a production OLTP database into analytics, ML feature stores, and lower environments?](<../../../PRACTICE/05_Data_Governance_and_Security/Governance/Stewardship/Q-11548_What_governance_controls_would_you_put_in_place_so_that_PII_.md>)
- [Deep] [How would you approach de-identification versus pseudonymization versus anonymization and where do you draw the line for analytics access to sensitive fields?](<../../../PRACTICE/05_Data_Governance_and_Security/Governance/Stewardship/Q-11563_How_would_you_approach_de-identification_versus_pseudonymiza.md>)
- [Deep] [Why not enforce all governance at the application layer, and what do you gain by pushing policy enforcement into the database or query engine itself?](<../../../PRACTICE/05_Data_Governance_and_Security/Governance/Stewardship/Q-11577_Why_not_enforce_all_governance_at_the_application_layer_and_.md>)
- [Deep] [How would you set up an automated PII discovery and classification scan across a large heterogeneous estate and manage the false positives it produces?](<../../../PRACTICE/05_Data_Governance_and_Security/Governance/Stewardship/Q-11592_How_would_you_set_up_an_automated_PII_discovery_and_classifi.md>)
- [Deep] [What is your approach to governing the risk that BI dashboards silently diverge because each analyst re-implements the same metric with slightly different SQL?](<../../../PRACTICE/05_Data_Governance_and_Security/Governance/Stewardship/Q-11630_What_is_your_approach_to_governing_the_risk_that_BI_dashboar.md>)
- [Deep] [How do you govern the emergency-access break-glass procedure to sensitive data during an incident so it's usable under pressure yet fully audited afterward?](<../../../PRACTICE/05_Data_Governance_and_Security/Governance/Stewardship/Q-11641_How_do_you_govern_the_emergency-access_break-glass_procedure.md>)
- [Expert] [What metrics would you present to the board to prove a data governance program is delivering value rather than just adding cost and friction?](<../../../PRACTICE/05_Data_Governance_and_Security/Governance/Stewardship/Q-11558_What_metrics_would_you_present_to_the_board_to_prove_a_data_.md>)
- [Expert] [How do you build an audit trail for who accessed which sensitive rows and when, across systems, that is tamper-evident and query-efficient at petabyte scale?](<../../../PRACTICE/05_Data_Governance_and_Security/Governance/Stewardship/Q-11578_How_do_you_build_an_audit_trail_for_who_accessed_which_sensi.md>)

## Related concepts

- [Policy as Code for Data](<../Data_Strategy/Policy_as_Code_for_Data.md>) *(Data Strategy)*
- [Data Ownership](<../Lineage/Data_Ownership.md>) *(Lineage)*
- [Access Governance for LLM Pipelines](<../../Security_and_Privacy/Access_Governance/Access_Governance_for_LLM_Pipelines.md>) *(Access Governance)*

## References

- *Building an Anonymization Pipeline* — Luk Arbuckle, Khaled El Emam — Introduction
- *Building an Anonymization Pipeline* — Luk Arbuckle, Khaled El Emam — A Practical Risk-Management Framework
- *Building an Anonymization Pipeline* — Luk Arbuckle, Khaled El Emam — Identified Data
- *Data Governance with Unity Catalog on Databricks* — Kiran Sreekumar, Karthik Subbarao — Being Compliant with Regulatory Standards
- *Practical Data Privacy* — Katharine Jarmul — Data Governance and Simple Privacy Approaches

