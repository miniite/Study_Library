---
id: "04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Centralized_vs_Mesh_Observability_Data"
level: 5
type: "deep-dive"
topic: "04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture"
tools: ["AWS"]
---

# Centralized vs Mesh Observability Data

**L5 Expert** · [↑ Data Architecture](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [AWS](<../../../../TOOLS/AWS/_index.md>)

## Practice questions

- [Deep] [How does Conway's law shape whether a central feature platform or a mesh of feature products succeeds at your org? What team-topology changes must precede the technical migration to decentralized ML data ownership?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10790_How_does_Conway_s_law_shape_whether_a_central_feature_platfo.md>)
- [Deep] [When is a dimensional warehouse still right for observability analytics (SLO reporting, MTTR trends) versus a wide event store versus a mesh of team telemetry products? What determined your answer at your last org?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10881_When_is_a_dimensional_warehouse_still_right_for_observabilit.md>)
- [Deep] [Argue whether the observability warehouse should ever be system-of-record for anything, or purely a derived copy of the event stream. What goes wrong when teams start treating a rolled-up telemetry table as authoritative for capacity or billing decisions?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10898_Argue_whether_the_observability_warehouse_should_ever_be_sys.md>)
- [Expert] [Your telemetry estate ingests 20PB/day of metrics logs traces and events. Architect the org-wide observability data platform: what lives in a hot columnar store versus a cheap object-store lake versus a warehouse for post-hoc analytics, and how do you draw the tiering boundary by query pattern?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10845_Your_telemetry_estate_ingests_20PB_day_of_metrics_logs_trace.md>)
- [Expert] [Define RTO and RPO for the observability data estate itself. If the telemetry pipeline goes down during an outage, you're blind. Architect DR so the monitoring plane survives a region loss, and explain why observability DR differs from OLTP DR.](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10870_Define_RTO_and_RPO_for_the_observability_data_estate_itself_.md>)
- [Expert] [A domain team wants to own its telemetry pipeline end-to-end in a mesh but lacks data-engineering skill, and a broken pipeline blinds on-call. How do you resolve decentralized ownership against uneven maturity, and what must the paved observability road provide for free?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10879_A_domain_team_wants_to_own_its_telemetry_pipeline_end-to-end.md>)
- [Expert] [You're evaluating a 'mesh-lite': a product-catalog layer over your existing central observability platform so teams own their telemetry data products but the platform stays central. Is this a pragmatic hybrid or worst-of-both, and where does it break down at scale?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10901_You_re_evaluating_a_mesh-lite_a_product-catalog_layer_over_y.md>)
- [Expert] [Compare self-managing your own Iceberg-on-S3 catalog and compute for telemetry versus a managed catalog (Unity/Polaris/Glue). What in-house capabilities must the observability platform team have before self-managing is defensible at petabyte scale?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10923_Compare_self-managing_your_own_Iceberg-on-S3_catalog_and_com.md>)

## Related concepts

- [Mesh vs Central Observability Lake](<../../Observability_Data_Lake/Mesh_vs_Central_Observability_Lake.md>) *(Observability Data Lake)*
- [Centralized vs Federated Telemetry](<../../../../05_Data_Governance_and_Security/Governance/Data_Strategy/Centralized_vs_Federated_Telemetry.md>) *(Data Strategy)*
- [Cardinality Isolation](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Provisional_and_Final_Results/Cardinality_Isolation.md>) *(Streaming Architecture)*

## References

- *97 Things Every Data Engineer Should Know* — Tobias Macey (ed.) — Data Observability The Next Frontier of Data Engineering

