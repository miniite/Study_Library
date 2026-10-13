---
id: "05_Data_Governance_and_Security/Governance/Catalog_and_Metadata/Catalog_Consistency_Model"
level: 5
type: "concept"
topic: "05_Data_Governance_and_Security/Governance/Catalog_and_Metadata"
tools: ["AWS", "Apache_Kafka"]
---

# Catalog Consistency Model

**L5 Expert** · [↑ Catalog and Metadata](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [AWS](<../../../TOOLS/AWS/_index.md>)
- [Apache Kafka](<../../../TOOLS/Apache_Kafka/_index.md>)

## Practice questions

- [Deep] [How do you keep the catalog and the actual data estate from drifting — detecting orphaned entries, tables that exist but were never ingested, and ghost lineage to deleted assets — as a continuous reconciliation process?](<../../../PRACTICE/05_Data_Governance_and_Security/Governance/Catalog_and_Metadata/Q-11149_How_do_you_keep_the_catalog_and_the_actual_data_estate_from_.md>)
- [Deep] [How do you model metadata that changes at very different velocities in one system — schema (rare), freshness/row-counts (per-run), ownership (occasional), popularity (continuous) — without either a bloated event stream or a stale snapshot?](<../../../PRACTICE/05_Data_Governance_and_Security/Governance/Catalog_and_Metadata/Q-11164_How_do_you_model_metadata_that_changes_at_very_different_vel.md>)
- [Deep] [How would you use the catalog to power impact analysis before a schema change ships — telling the engineer 'this drop-column will break 12 dashboards and 3 ML models'? What has to be true about lineage coverage for that to be trustworthy?](<../../../PRACTICE/05_Data_Governance_and_Security/Governance/Catalog_and_Metadata/Q-11205_How_would_you_use_the_catalog_to_power_impact_analysis_befor.md>)
- [Deep] [Design how you'd handle soft-delete versus hard-delete of metadata — a table is dropped, then recreated a week later. Should lineage history persist, and how do you avoid resurrecting stale metadata or losing audit trail?](<../../../PRACTICE/05_Data_Governance_and_Security/Governance/Catalog_and_Metadata/Q-11238_Design_how_you_d_handle_soft-delete_versus_hard-delete_of_me.md>)
- [Expert] [Probe: you propose storing all metadata as immutable events (event-sourced). What are the operational downsides — replay time, storage growth, snapshotting — and when would you regret not just mutating a relational store?](<../../../PRACTICE/05_Data_Governance_and_Security/Governance/Catalog_and_Metadata/Q-11176_Probe_you_propose_storing_all_metadata_as_immutable_events_e.md>)
- [Expert] [How do you handle metadata versioning and time-travel — answering 'what did this table's schema and its lineage look like on the day of that incident three months ago'?](<../../../PRACTICE/05_Data_Governance_and_Security/Governance/Catalog_and_Metadata/Q-11197_How_do_you_handle_metadata_versioning_and_time-travel_answer.md>)
- [Expert] [How would you architect metadata capture so it survives an engine migration — say Redshift to Snowflake to Databricks over several years — without re-implementing lineage extraction each time?](<../../../PRACTICE/05_Data_Governance_and_Security/Governance/Catalog_and_Metadata/Q-11211_How_would_you_architect_metadata_capture_so_it_survives_an_e.md>)
- [Expert] [Probe: your metadata event schema needs to evolve as you add features. How do you version the metadata model itself and migrate millions of existing entities without downtime or breaking consumers of the metadata API?](<../../../PRACTICE/05_Data_Governance_and_Security/Governance/Catalog_and_Metadata/Q-11222_Probe_your_metadata_event_schema_needs_to_evolve_as_you_add_.md>)

## Related concepts

- [Lineage for Dynamic SQL](<../Lineage/Lineage_for_Dynamic_SQL.md>) *(Lineage)*
- [Distinguishing Drift from Pipeline Breaks](<../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Distinguishing_Drift_from_Pipeline_Breaks.md>) *(Data Architecture)*
- [Glossary Metadata and Contracts](<../Lineage/Glossary_Metadata_and_Contracts.md>) *(Lineage)*

