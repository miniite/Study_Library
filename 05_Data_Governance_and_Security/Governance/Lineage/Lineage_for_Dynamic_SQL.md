---
id: "05_Data_Governance_and_Security/Governance/Lineage/Lineage_for_Dynamic_SQL"
level: 4
type: "concept"
topic: "05_Data_Governance_and_Security/Governance/Lineage"
tools: []
---

# Lineage for Dynamic SQL

**L4 Advanced** · [↑ Lineage](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Practice questions

- [Intermediate] [How did you integrate OpenLineage into your Airflow DAGs? What events does it emit and where do you store them?](<../../../PRACTICE/05_Data_Governance_and_Security/Governance/Lineage/Q-11444_How_did_you_integrate_OpenLineage_into_your_Airflow_DAGs_Wha.md>)
- [Intermediate] [What is the cost of NOT having lineage during a production incident? Give a concrete before/after from your experience.](<../../../PRACTICE/05_Data_Governance_and_Security/Governance/Lineage/Q-11506_What_is_the_cost_of_NOT_having_lineage_during_a_production_i.md>)
- [Deep] [What is the difference between design-time and runtime lineage, and when has each mattered in an incident you handled?](<../../../PRACTICE/05_Data_Governance_and_Security/Governance/Lineage/Q-11446_What_is_the_difference_between_design-time_and_runtime_linea.md>)
- [Deep] [Why is lineage granularity a cost/benefit trade-off? When would column-level lineage be overkill?](<../../../PRACTICE/05_Data_Governance_and_Security/Governance/Lineage/Q-11460_Why_is_lineage_granularity_a_cost_benefit_trade-off_When_wou.md>)
- [Deep] [How would you catch and prevent a pipeline silently dropping rows so lineage shows a link but data completeness is wrong?](<../../../PRACTICE/05_Data_Governance_and_Security/Governance/Lineage/Q-11482_How_would_you_catch_and_prevent_a_pipeline_silently_dropping.md>)
- [Deep] [What's the difference between provenance and lineage, and why does the distinction matter for reproducibility?](<../../../PRACTICE/05_Data_Governance_and_Security/Governance/Lineage/Q-11508_What_s_the_difference_between_provenance_and_lineage_and_why.md>)
- [Deep] [What is the blast radius of a bad upstream change, and how do you communicate it to affected teams automatically using lineage?](<../../../PRACTICE/05_Data_Governance_and_Security/Governance/Lineage/Q-11537_What_is_the_blast_radius_of_a_bad_upstream_change_and_how_do.md>)
- [Expert] [In a data mesh, how do you keep lineage consistent when each domain publishes its own data products independently?](<../../../PRACTICE/05_Data_Governance_and_Security/Governance/Lineage/Q-11481_In_a_data_mesh_how_do_you_keep_lineage_consistent_when_each_.md>)

## Related concepts

- [Scaling Lineage Graphs](<../Catalog_and_Metadata/Scaling_Lineage_Graphs.md>) *(Catalog and Metadata)*
- [Catalog Consistency Model](<../Catalog_and_Metadata/Catalog_Consistency_Model.md>) *(Catalog and Metadata)*
- [Distinguishing Drift from Pipeline Breaks](<../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Distinguishing_Drift_from_Pipeline_Breaks.md>) *(Data Architecture)*

## References

- *97 Things Every Data Engineer Should Know* — Tobias Macey (ed.) — The Importance of Data Lineage

