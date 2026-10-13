---
id: "04_Data_Architecture_and_Strategy/Architecture_Patterns/Observability_Data_Lake/Exactly_Once_into_Observability_Lakes"
level: 5
type: "concept"
topic: "04_Data_Architecture_and_Strategy/Architecture_Patterns/Observability_Data_Lake"
tools: ["Apache_Kafka", "Apache_Flink"]
---

# Exactly Once into Observability Lakes

**L5 Expert** · [↑ Observability Data Lake](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Apache Kafka](<../../../TOOLS/Apache_Kafka/_index.md>)
- [Apache Flink](<../../../TOOLS/Apache_Flink/_index.md>)

## Practice questions

- [Deep] [What is your strategy for exactly-once versus at-least-once delivery from ingest to lake and why does it matter for alert counting accuracy?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Observability_Data_Lake/Q-34426_What_is_your_strategy_for_exactly-once_versus_at-least-once_.md>)
- [Deep] [How do you design incremental materialized views over the lake for the top 50 alert queries without re-scanning everything each run?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Observability_Data_Lake/Q-34440_How_do_you_design_incremental_materialized_views_over_the_la.md>)
- [Deep] [How would you decide whether to precompute anomaly features in the lake batch layer or compute them on the fly in a streaming job?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Observability_Data_Lake/Q-34455_How_would_you_decide_whether_to_precompute_anomaly_features_.md>)
- [Deep] [Explain the trade-off between storing pre-computed correlation edges in the lake versus computing them at query time during an incident.](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Observability_Data_Lake/Q-34472_Explain_the_trade-off_between_storing_pre-computed_correlati.md>)
- [Deep] [How would you architect the lake so a postmortem can time-travel to the exact table state at the incident minute while a compaction job ran hours ago?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Observability_Data_Lake/Q-34546_How_would_you_architect_the_lake_so_a_postmortem_can_time-tr.md>)
- [Deep] [What is your framework for deciding which telemetry gets a strict typed schema versus schema-on-read flexibility given hundreds of heterogeneous vendor log sources?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Observability_Data_Lake/Q-34565_What_is_your_framework_for_deciding_which_telemetry_gets_a_s.md>)
- [Deep] [How do you decide whether to invest in a graph layer over the lake for service-topology and dependency-aware correlation versus keeping it relational?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Observability_Data_Lake/Q-34582_How_do_you_decide_whether_to_invest_in_a_graph_layer_over_th.md>)
- [Expert] [How would you architect right-to-erasure and GDPR compliance on an immutable append-only lake without a rewrite storm every time a deletion request lands?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Observability_Data_Lake/Q-34537_How_would_you_architect_right-to-erasure_and_GDPR_compliance.md>)

## Related concepts

- [Data Quality Monitoring](<../../../02_Data_Engineering/Data_Quality/Data_Observability/Data_Quality_Monitoring.md>) *(Data Observability)*
- [Real-time OLAP Serving Layer](<../Data_Architecture/Tiered_Lake_Storage/Real-time_OLAP_Serving_Layer.md>) *(Data Architecture)*
- [Reproducible Telemetry State](<../Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Reproducible_Telemetry_State.md>) *(Data Architecture)*

