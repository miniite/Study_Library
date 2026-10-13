---
id: "03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Duplicates_During_Rebalance"
level: 4
type: "concept"
topic: "03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture"
tools: ["Apache_Kafka", "Apache_Flink"]
---

# Duplicates During Rebalance

**L4 Advanced** · [↑ Streaming Architecture](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Apache Kafka](<../../../TOOLS/Apache_Kafka/_index.md>)
- [Apache Flink](<../../../TOOLS/Apache_Flink/_index.md>)

## Practice questions

- [Intermediate] [Design a streaming pipeline where a human can acknowledge an alert and that ack flows back to suppress duplicate pages for the same incident. How does the ack event propagate and how long does suppression last?](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Q-38213_Design_a_streaming_pipeline_where_a_human_can_acknowledge_an.md>)
- [Deep] [Walk me through how you'd design a streaming deduplication stage for alerts so the same underlying fault firing across 200 hosts collapses into one incident. What state and what window do you use?](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Q-38133_Walk_me_through_how_you_d_design_a_streaming_deduplication_s.md>)
- [Deep] [Your remediation service consumes a stream of 'anomaly-confirmed' events and triggers a runbook. How do you make that consumer idempotent so a Kafka redelivery doesn't restart the same service twice?](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Q-38135_Your_remediation_service_consumes_a_stream_of_anomaly-confir.md>)
- [Deep] [Your session-window logic groups related alerts into an incident by a 5-minute inactivity gap. For flapping alerts, sessions never close. How would you make the gap adaptive or cap the window?](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Q-38142_Your_session-window_logic_groups_related_alerts_into_an_inci.md>)
- [Deep] [Your streaming alert pipeline occasionally emits a burst of duplicate pages during a Kafka rebalance. Explain why rebalances cause reprocessing and how you'd make the pipeline resilient to it.](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Q-38160_Your_streaming_alert_pipeline_occasionally_emits_a_burst_of_.md>)
- [Deep] [Your streaming remediation flow auto-restarts unhealthy pods based on a stream of health events. How do you add a circuit breaker so a bad event storm doesn't trigger a restart loop across the fleet?](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Q-38167_Your_streaming_remediation_flow_auto-restarts_unhealthy_pods.md>)
- [Deep] [Your team accidentally reset consumer offsets to earliest on the live anomaly pipeline and it re-alerted on a week of old data. What went wrong, and how do you design a safe replay path that never touches the live consumer?](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Q-38180_Your_team_accidentally_reset_consumer_offsets_to_earliest_on.md>)
- [Deep] [Your streaming aggregation for anomaly detection over-counts during a partition failover because uncommitted state got reprocessed. Explain how transactions or idempotent updates prevent that double-count.](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Q-38188_Your_streaming_aggregation_for_anomaly_detection_over-counts.md>)

## Related concepts

- [Pipeline Design Patterns](<../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Design/Pipeline_Design_Patterns.md>) *(Pipeline Design)*
- [Streaming Correctness and Drift](<../../Distributed_Processing/Batch_vs_Stream/Streaming_Correctness_and_Drift.md>) *(Batch vs Stream)*
- [Gossip and Deduplication](<../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Clock_Skew_and_Time_Windows/Gossip_and_Deduplication.md>) *(Distributed Systems Core)*

