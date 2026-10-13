---
id: "04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Central_vs_Domain_Feature_Modeling"
level: 5
type: "deep-dive"
topic: "04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture"
tools: ["AWS", "Apache_Spark", "Apache_Kafka", "Python"]
---

# Central vs Domain Feature Modeling

**L5 Expert** · [↑ Data Architecture](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [AWS](<../../../../TOOLS/AWS/_index.md>)
- [Apache Spark](<../../../../TOOLS/Apache_Spark/_index.md>)
- [Apache Kafka](<../../../../TOOLS/Apache_Kafka/_index.md>)
- [Python](<../../../../TOOLS/Python/_index.md>)

## Practice questions

- [Intermediate] [Explain what a data contract is and what you, as a downstream ML consumer, would insist it guarantees about a feature source.](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10696_Explain_what_a_data_contract_is_and_what_you_as_a_downstream.md>)
- [Deep] [Your model's input table is defined by a dbt model owned by analytics engineering. How do you coordinate so their refactor doesn't silently break your features?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10702_Your_model_s_input_table_is_defined_by_a_dbt_model_owned_by_.md>)
- [Deep] [How do you architect a lake to serve both interactive BI (sub-second) and heavy ML training on the same underlying data without either workload degrading the other? What layers/materializations are required?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10601_How_do_you_architect_a_lake_to_serve_both_interactive_BI_sub.md>)
- [Deep] [How do you architect cost attribution for shared training compute (one Spark/GPU-adjacent cluster serving many model teams) so chargeback is fair and non-gameable? What query/job tagging makes per-model accounting possible?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10779_How_do_you_architect_cost_attribution_for_shared_training_co.md>)
- [Deep] [How do you architect the boundary between the operational plane (event log / OLTP) and the analytical/feature plane so ML consumption doesn't couple them into a distributed monolith? Where does CDC-driven feature freshness pull the two too tightly together?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10820_How_do_you_architect_the_boundary_between_the_operational_pl.md>)
- [Deep] [When is a Kimball-style dimensional model still the right shape for ML feature serving versus a wide denormalized 'one big table' versus a mesh of domain feature products? What determined your answer at your last org?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10834_When_is_a_Kimball-style_dimensional_model_still_the_right_sh.md>)
- [Deep] [Explain how 'data mesh vs data fabric' plays out for ML: are automated metadata-driven feature integration (fabric) and decentralized feature ownership (mesh) competing or complementary in your target architecture? Where does each sit?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10844_Explain_how_data_mesh_vs_data_fabric_plays_out_for_ML_are_au.md>)
- [Expert] [Design the org-wide contract that guarantees the exact feature values used in training are byte-identical to those served online, across a lakehouse offline store and a low-latency online store. What makes this a data-architecture problem rather than an ML one?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10746_Design_the_org-wide_contract_that_guarantees_the_exact_featu.md>)

## Related concepts

- [Training Data Provenance](<../../../../05_Data_Governance_and_Security/Governance/Data_Strategy/Training_Data_Provenance.md>) *(Data Strategy)*
- [Data Vault](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Modeling_Methodologies/Data_Vault.md>) *(Modeling Methodologies)*
- [Multi-cloud Strategy](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Managed_Services_vs_Self-hosting/Multi-cloud_Strategy.md>) *(Cloud Fundamentals)*

## References

- *97 Things Every Data Engineer Should Know* — Tobias Macey (ed.) — Embracing Data Silos The Journey Through a Fragmented Data World
- *Building Data Integration Solutions* — Jay Borthen — Models, Architectures, Methods, and Patterns
- *Building Knowledge Graphs* — Jesús Barrasa, Maya Natarajan, Jim Webber — From Knowledge Graphs to Knowledge Lakes
- *Data Engineering Fundamentals* — Zhaolong Liu — CHAPTER 1Understanding Data Engineering
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Serving Data for Analytics, Machine Learning, and Reverse ETL
- *Hands-On Healthcare Data* — Andrew Nguyen — Machine Learning and Analytics
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Machine Learning with MLlib

