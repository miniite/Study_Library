---
id: "07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Clock_Skew_and_Time_Windows"
level: 4
type: "concept"
topic: "07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core"
tools: []
---

# Clock Skew and Time Windows

**L4 Advanced** · [↑ Distributed Systems Core](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Go deeper

- [Keeping Indexes in Sync via CDC](<Keeping_Indexes_in_Sync_via_CDC.md>)
- [Distributed Tracing](<Distributed_Tracing.md>)
- [Gossip and Deduplication](<Gossip_and_Deduplication.md>)
- [Distributed Backfills](<Distributed_Backfills.md>)
- [SLO Burn Rate Alerting](<SLO_Burn_Rate_Alerting.md>)

## Practice questions

- [Intermediate] [Your online feature store and offline warehouse disagree for some entities during an incident, causing train-serve skew. Explain how you'd detect the skew in production and reconcile without taking the serving path down.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23185_Your_online_feature_store_and_offline_warehouse_disagree_for.md>)
- [Deep] [A real-time feature counts events in a 5-minute window and is served at inference, but a late or duplicate event corrupts the count. Explain the windowing and exactly-once-versus-at-least-once concern and how event-time watermarking keeps the served feature correct.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23186_A_real-time_feature_counts_events_in_a_5-minute_window_and_i.md>)
- [Deep] [Your NTP/chrony drift alarm fired and shortly after a Cassandra last-write-wins column silently lost data. Explain the operational link between clock discipline and correctness and how you'd monitor drift as a first-class SLI.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23447_Your_NTP_chrony_drift_alarm_fired_and_shortly_after_a_Cassan.md>)
- [Deep] [Explain how clock skew across your training and labeling systems can corrupt a time-windowed training set (e.g. features from 'before' the label actually being after). How would you defend against it with event-time rather than processing-time semantics?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23658_Explain_how_clock_skew_across_your_training_and_labeling_sys.md>)
- [Deep] [Your incident-timeline reconstruction relies on ordering events from three services whose clocks disagree by 400ms. Explain how you'd use logical or hybrid-logical timestamps stamped at ingest to reconstruct a defensible causal order instead of trusting wall clocks.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23867_Your_incident-timeline_reconstruction_relies_on_ordering_eve.md>)
- [Deep] [Your anomaly model reads a real-time feature (error-rate over the last minute) from a stream, and a duplicate or late event inflated the rate into a false positive. Explain the exactly-once-versus-at-least-once concern and how event-time watermarking keeps the served feature correct.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23908_Your_anomaly_model_reads_a_real-time_feature_error-rate_over.md>)
- [Deep] [Explain how clock skew across your event producers can corrupt a time-windowed aggregation so events that are 'before' actually land after. How do you enforce event-time semantics and monitor drift as a correctness SLI?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-24330_Explain_how_clock_skew_across_your_event_producers_can_corru.md>)
- [Deep] [You're the principal on an AIOps program where the anomaly model's own training data comes from a distributed feature pipeline that lagged, so it learned on stale telemetry and started false-alarming. Explain the freshness SLI you'd put on the feature path and why staleness here is a correctness bug, not a cost one.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23959_You_re_the_principal_on_an_AIOps_program_where_the_anomaly_m.md>)

## Related concepts

- [Event-time Watermarks](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Provisional_and_Final_Results/Event-time_Watermarks.md>) *(Streaming Architecture)*
- [Distribution Change Alerts](<../../../../02_Data_Engineering/Data_Quality/Data_Observability/Distribution_Change_Alerts.md>) *(Data Observability)*
- [Unified Batch and Streaming Semantics](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream/Batch_and_Stream_Reconciliation/Unified_Batch_and_Streaming_Semantics.md>) *(Batch vs Stream)*

## References

- *Data Engineering Design Patterns* — Bartosz Konieczny — Error Management Design Patterns › Problem
- *Data Engineering Design Patterns* — Bartosz Konieczny — Error Management Design Patterns › Max strategy and event skew
- *Data Engineering Design Patterns* — Bartosz Konieczny — Pattern: Skew Detector
- *Data Engineering Design Patterns* — Bartosz Konieczny — Time Detectors › Consequences
- *Data Contracts: Developing Production-Grade Pipelines at Scale* — Chad Sanderson, Mark Freeman — The Data Contract Components Data Assets and Contract Definition › Event Sourcing and Event Streams
- *Data Contracts: Developing Production-Grade Pipelines at Scale* — Chad Sanderson, Mark Freeman — Shift Left The Cultural Change Needed for Data Contracts › What Needs to Be True: What We Missed About Shift Left Data
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Aggregations with Event-Time Windows › Semantic guarantees with watermarks
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Arbitrary Stateful Computations › Event-time timeouts
- *Designing Data-Intensive Applications* — Martin Kleppmann — Reasoning About Time
- *Designing Data-Intensive Applications* — Martin Kleppmann — Reasoning About Time › Event time versus processing time
- *Designing Data-Intensive Applications* — Martin Kleppmann — Reasoning About Time › Knowing when you're ready
- *Designing Data-Intensive Applications* — Martin Kleppmann — B
- *Think Distributed Systems* — Dominik Tornow — System models, order, and time › No notion of time
- *Think Distributed Systems* — Dominik Tornow — System models, order, and time › 2.2.1 The happened-before relationship
- *Think Distributed Systems* — Dominik Tornow — System models, order, and time › Mitigation
- *Database Internals* — Alex Petrov — Introduction and Overview › Clocks and Time
- *Database Internals* — Alex Petrov — Replication and Consistency › Vector clocks

