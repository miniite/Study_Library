---
id: "04_Data_Architecture_and_Strategy/Platform_Strategy/Master_Data_Management/MDM_for_Batch_and_Streaming"
level: 5
type: "concept"
topic: "04_Data_Architecture_and_Strategy/Platform_Strategy/Master_Data_Management"
tools: ["Apache_Kafka"]
---

# MDM for Batch and Streaming

**L5 Expert** · [↑ Master Data Management](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Apache Kafka](<../../../TOOLS/Apache_Kafka/_index.md>)

## Practice questions

- [Deep] [How do you measure and defend MDM data quality — what metrics (match rate, false-merge rate, completeness, uniqueness, stewardship backlog) do you put on an exec dashboard and what SLAs do you commit?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Platform_Strategy/Master_Data_Management/Q-32000_How_do_you_measure_and_defend_MDM_data_quality_what_metrics_.md>)
- [Deep] [Explain how you'd instrument observability for the MDM hub — what do you alert on (match drift, steward backlog growth, publish lag, survivorship rule error rate) versus what you dashboard?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Platform_Strategy/Master_Data_Management/Q-32047_Explain_how_you_d_instrument_observability_for_the_MDM_hub_w.md>)
- [Deep] [What's your view on serving master data via an API/GraphQL 'entity service' vs publishing to a shared data product in the lakehouse — when is each the right consumption pattern?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Platform_Strategy/Master_Data_Management/Q-32060_What_s_your_view_on_serving_master_data_via_an_API_GraphQL_e.md>)
- [Expert] [How do you architect real-time / streaming MDM — resolving and enriching an entity within an event pipeline at sub-second latency — and where does batch reconciliation still have to backstop it?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Platform_Strategy/Master_Data_Management/Q-31998_How_do_you_architect_real-time_streaming_MDM_resolving_and_e.md>)
- [Expert] [Describe the failure modes when MDM becomes a synchronous dependency in the write path of an order/payment flow — how do you keep it out of the critical path while still enforcing uniqueness?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Platform_Strategy/Master_Data_Management/Q-32011_Describe_the_failure_modes_when_MDM_becomes_a_synchronous_de.md>)
- [Expert] [How do you architect for the case where MDM must serve both low-latency operational lookups (single entity by key) and heavy analytical match jobs — one store, CQRS, or two stores, and why?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Platform_Strategy/Master_Data_Management/Q-32029_How_do_you_architect_for_the_case_where_MDM_must_serve_both_.md>)
- [Expert] [How would you architect MDM to feed a real-time personalization/decisioning engine where a stale identity means a bad recommendation — what's the freshness contract and fallback?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Platform_Strategy/Master_Data_Management/Q-32046_How_would_you_architect_MDM_to_feed_a_real-time_personalizat.md>)
- [Expert] [Interviewer probe: 'If MDM is so central, isn't it a single point of failure?' Walk through your HA/DR design and RTO/RPO targets for the hub and its event streams.](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Platform_Strategy/Master_Data_Management/Q-32065_Interviewer_probe_If_MDM_is_so_central_isn_t_it_a_single_poi.md>)

## Related concepts

- [Data Vault](<../../../02_Data_Engineering/Modeling_and_Warehousing/Modeling_Methodologies/Data_Vault.md>) *(Modeling Methodologies)*
- [Warehouse Vendor Exit Strategy](<../../Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Warehouse_Vendor_Exit_Strategy.md>) *(Data Architecture)*
- [Platform-level Data Quality](<../Data_Platform_Strategy/Platform-level_Data_Quality.md>) *(Data Platform Strategy)*

