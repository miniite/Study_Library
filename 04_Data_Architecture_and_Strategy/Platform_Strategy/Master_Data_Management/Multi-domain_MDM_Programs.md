---
id: "04_Data_Architecture_and_Strategy/Platform_Strategy/Master_Data_Management/Multi-domain_MDM_Programs"
level: 5
type: "concept"
topic: "04_Data_Architecture_and_Strategy/Platform_Strategy/Master_Data_Management"
tools: []
---

# Multi-domain MDM Programs

**L5 Expert** · [↑ Master Data Management](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Practice questions

- [Deep] [An interviewer pushes back: 'Why not just do dbt models and dedupe in the warehouse — why buy MDM at all?' How do you respond and where does that approach actually break?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Platform_Strategy/Master_Data_Management/Q-31988_An_interviewer_pushes_back_Why_not_just_do_dbt_models_and_de.md>)
- [Deep] [Contrast MDM with the modern 'Customer Data Platform' — a stakeholder wants to buy a CDP instead of doing MDM; where do they overlap, where do they diverge, and what would you tell them?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Platform_Strategy/Master_Data_Management/Q-32009_Contrast_MDM_with_the_modern_Customer_Data_Platform_a_stakeh.md>)
- [Deep] [Explain the difference between operational MDM and analytical MDM in your architecture, why some orgs deliberately run both, and how you keep them from drifting apart.](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Platform_Strategy/Master_Data_Management/Q-32038_Explain_the_difference_between_operational_MDM_and_analytica.md>)
- [Deep] [How do you reconcile MDM survivorship with 'source of truth by attribute' governance — some attributes are authoritative in CRM, others in ERP — how is that policy encoded and enforced?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Platform_Strategy/Master_Data_Management/Q-32054_How_do_you_reconcile_MDM_survivorship_with_source_of_truth_b.md>)
- [Expert] [Design a multi-domain MDM program covering customer, product, supplier, and location — how do you sequence domains, and why is customer usually not the one you should start with?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Platform_Strategy/Master_Data_Management/Q-31987_Design_a_multi-domain_MDM_program_covering_customer_product_.md>)
- [Expert] [What's your approach to hierarchy management for organizations (legal entity vs billing vs household vs corporate family tree), and how do you version hierarchies that change through M&A?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Platform_Strategy/Master_Data_Management/Q-31997_What_s_your_approach_to_hierarchy_management_for_organizatio.md>)
- [Expert] [How do you handle product master data (PIM) where the same SKU has different attributes per channel/region — is that one master with variants, many masters, or a different pattern entirely?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Platform_Strategy/Master_Data_Management/Q-32027_How_do_you_handle_product_master_data_PIM_where_the_same_SKU.md>)
- [Expert] [Describe how you'd design MDM for a marketplace with buyers and sellers who are sometimes the same physical entity across sides — what identity model prevents policy loopholes and fraud?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Platform_Strategy/Master_Data_Management/Q-32042_Describe_how_you_d_design_MDM_for_a_marketplace_with_buyers_.md>)

## Related concepts

- [Data Vault](<../../../02_Data_Engineering/Modeling_and_Warehousing/Modeling_Methodologies/Data_Vault.md>) *(Modeling Methodologies)*
- [Cross-domain Joins in a Mesh](<../../Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Cross-domain_Joins_in_a_Mesh.md>) *(Data Architecture)*
- [Platform-level Data Quality](<../Data_Platform_Strategy/Platform-level_Data_Quality.md>) *(Data Platform Strategy)*

