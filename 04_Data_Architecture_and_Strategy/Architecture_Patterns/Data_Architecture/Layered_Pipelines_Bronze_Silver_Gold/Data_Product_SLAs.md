---
id: "04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Data_Product_SLAs"
level: 5
type: "deep-dive"
topic: "04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture"
tools: ["AWS", "Apache_Kafka", "Prometheus"]
---

# Data Product SLAs

**L5 Expert** · [↑ Data Architecture](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [AWS](<../../../../TOOLS/AWS/_index.md>)
- [Apache Kafka](<../../../../TOOLS/Apache_Kafka/_index.md>)
- [Prometheus](<../../../../TOOLS/Prometheus/_index.md>)

## Practice questions

- [Intermediate] [Your team wants to publish an orders data product with an SLA. What would you actually promise consumers about freshness, availability, and correctness, and how do you measure whether you're meeting it?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-11090_Your_team_wants_to_publish_an_orders_data_product_with_an_SL.md>)
- [Deep] [A data product you consume publishes a 99.9% freshness SLA but your model needs fresher data. How do you handle that mismatch without owning their pipeline?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10723_A_data_product_you_consume_publishes_a_99_9_freshness_SLA_bu.md>)
- [Deep] [Explain how you'd architect feature freshness SLAs that compose across a chain: raw ingestion to silver to feature to online store to model. Where does a single flaky upstream silently violate a model's freshness guarantee, and who's paged?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10760_Explain_how_you_d_architect_feature_freshness_SLAs_that_comp.md>)
- [Deep] [A domain insists its feature product is 'done' after one publish, but ML feature products need continuous operation. How do you architect and fund the ongoing operational model (on-call, freshness SLAs, deprecation) so feature products don't silently rot?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10838_A_domain_insists_its_feature_product_is_done_after_one_publi.md>)
- [Deep] [How do you architect a 'data product SLO' framework distinct from data quality: what availability, freshness, and correctness SLIs do you standardize across a mesh, how do you set error budgets on data products, and what does an SLO breach trigger operationally?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-11030_How_do_you_architect_a_data_product_SLO_framework_distinct_f.md>)
- [Expert] [A domain publishes a data product with a 99.9% freshness SLA but a critical downstream product needs 99.99%. How do you handle SLA composition and dependency risk across a chain of data products in a mesh?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10596_A_domain_publishes_a_data_product_with_a_99_9_freshness_SLA_.md>)
- [Expert] [How do you set RTO/RPO for the ML data estate and design DR so a region loss doesn't take out feature serving for a fraud model? Why is DR for online feature stores fundamentally harder than for the offline lake?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10773_How_do_you_set_RTO_RPO_for_the_ML_data_estate_and_design_DR_.md>)
- [Expert] [A telemetry data product publishes a 99.9% freshness SLA but a critical auto-remediation flow needs 99.99%. Architect SLA composition and dependency risk across a chain of telemetry products in a mesh so the automation team isn't held hostage by a weak upstream.](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10903_A_telemetry_data_product_publishes_a_99_9_freshness_SLA_but_.md>)

## Related concepts

- [Quality SLAs and Rule Testing](<../../../../02_Data_Engineering/Data_Quality/Quality_Frameworks_and_SLAs/Quality_SLAs_and_Rule_Testing.md>) *(Quality Frameworks and SLAs)*
- [Data Product Discoverability](<../../Data_Mesh/Data_Product_Discoverability.md>) *(Data Mesh)*

