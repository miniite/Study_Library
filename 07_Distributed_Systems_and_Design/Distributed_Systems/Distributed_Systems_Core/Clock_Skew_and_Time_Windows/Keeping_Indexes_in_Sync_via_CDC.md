---
id: "07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Keeping_Indexes_in_Sync_via_CDC"
level: 5
type: "deep-dive"
topic: "07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core"
tools: ["Apache_Kafka"]
---

# Keeping Indexes in Sync via CDC

**L5 Expert** · [↑ Distributed Systems Core](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Apache Kafka](<../../../../TOOLS/Apache_Kafka/_index.md>)

## Practice questions

- [Intermediate] [Explain how you'd keep a search index in sync with a primary DB via CDC and detect drift in production. What reconciliation job would you run and how do you avoid it becoming a load spike itself?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23460_Explain_how_you_d_keep_a_search_index_in_sync_with_a_primary.md>)
- [Deep] [Explain how you'd keep an Elasticsearch/OpenSearch index in sync with a Postgres source of truth using CDC. What at-least-once and ordering guarantees does log-based CDC give, and how do you make the index writes idempotent?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-24065_Explain_how_you_d_keep_an_Elasticsearch_OpenSearch_index_in_.md>)
- [Deep] [Explain how change data capture (CDC) works and why it's often better than polling for building derived data stores. What ordering and at-least-once guarantees does log-based CDC give you?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23268_Explain_how_change_data_capture_CDC_works_and_why_it_s_often.md>)
- [Deep] [Your RAG system's vector index is rebuilt nightly and swapped in. Explain how you'd make the index swap atomic across a distributed retrieval fleet so queries never hit a half-built index, and how you roll back a bad index build.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23684_Your_RAG_system_s_vector_index_is_rebuilt_nightly_and_swappe.md>)
- [Deep] [You keep a derived incident-store in sync with the raw alert stream via CDC, and it occasionally drifts so an incident shows fewer alerts than actually fired. Explain the ordering and at-least-once concerns of the CDC stream and the reconciliation job you'd run to detect and repair drift.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23901_You_keep_a_derived_incident-store_in_sync_with_the_raw_alert.md>)
- [Deep] [Explain how log-based CDC preserves ordering and at-least-once delivery from a source DB to a lakehouse and how you'd deduplicate and apply upserts/deletes in order so the derived table converges to the source.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-24287_Explain_how_log-based_CDC_preserves_ordering_and_at-least-on.md>)
- [Deep] [You keep a search index and a warehouse table in sync with the same source events but they drift over time. Explain how you'd detect drift with a periodic reconciliation and why exactly-once delivery alone doesn't guarantee two sinks stay identical.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-24290_You_keep_a_search_index_and_a_warehouse_table_in_sync_with_t.md>)
- [Deep] [Explain how you'd keep dozens of derived stores (search indexes, caches, read models) in sync with primaries via CDC and detect drift org-wide. Design the reconciliation and drift-alerting as a shared platform capability rather than each team reinventing it.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23601_Explain_how_you_d_keep_dozens_of_derived_stores_search_index.md>)

## Related concepts

- [CDC Fundamentals and Ordering](<../../../../02_Data_Engineering/Pipelines_and_Integration/Change_Data_Capture/CDC_Fundamentals_and_Ordering.md>) *(Change Data Capture)*
- [Index Strategy and Governance at Scale](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Index_Tuning_in_Practice/Index_Strategy_and_Governance_at_Scale.md>) *(Indexing)*

## References

- *Designing Data-Intensive Applications* — Martin Kleppmann — Q › R

