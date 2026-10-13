---
id: "TOOLS/Apache_Kafka/Kafka_Core/Kafka_Capacity_and_Cost"
level: 5
type: "deep-dive"
topic: "TOOLS/Apache_Kafka/Kafka_Core"
tools: ["Apache_Kafka"]
---

# Kafka Capacity and Cost

**L5 Expert** · [↑ Kafka Core](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Apache Kafka](<../../_index.md>)

## Practice questions

- [Intermediate] [How would you throttle a kafka-reassign-partitions run on a busy ingestion cluster so replication traffic doesn't starve live producers and stall the warehouse feed?](<../../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-31162_How_would_you_throttle_a_kafka-reassign-partitions_run_on_a_.md>)
- [Deep] [How does the rack-aware fetch-from-follower feature reduce cross-AZ cost for a consumer-heavy analytics workload, and what are its correctness caveats?](<../../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-31110_How_does_the_rack-aware_fetch-from-follower_feature_reduce_c.md>)
- [Expert] [What is cooperative sticky (incremental) rebalancing and how is it better than eager rebalancing?](<../../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-30603_What_is_cooperative_sticky_incremental_rebalancing_and_how_i.md>)
- [Expert] [Your cross-AZ replication traffic on a data-ingest Kafka cluster dominates the cloud bill; which levers (fetch-from-follower, compression, retention, RF) would you pull first and why?](<../../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-31109_Your_cross-AZ_replication_traffic_on_a_data-ingest_Kafka_clu.md>)
- [Expert] [You must set the org policy on one giant multi-tenant Kafka cluster versus many small clusters. Weigh cost operational leverage noisy-neighbor risk and regulatory isolation and give the decision rule you'd publish.](<../../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-30795_You_must_set_the_org_policy_on_one_giant_multi-tenant_Kafka_.md>)
- [Expert] [How do you architect crypto-shredding for GDPR erasure on Kafka: per-subject keys, envelope encryption, and key deletion instead of record deletion? Walk the trade-offs against replay compaction and long-term consumers.](<../../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-30812_How_do_you_architect_crypto-shredding_for_GDPR_erasure_on_Ka.md>)
- [Expert] [How do you design a fleet-wide idempotency and de-duplication strategy so every consumer that writes to an external store is exactly-once-effective, standardizing where dedupe state lives and how keys are derived, without forcing every team to reinvent it?](<../../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-30835_How_do_you_design_a_fleet-wide_idempotency_and_de-duplicatio.md>)
- [Expert] [An interviewer asks you to reason about tail latency at fleet scale: p99.9 produce latency correlates with GC pauses segment rolls and page-cache pressure. How do you architect the broker fleet (heap GC choice IO scheduling instance types) to bound tail latency as a platform guarantee?](<../../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-30846_An_interviewer_asks_you_to_reason_about_tail_latency_at_flee.md>)

## Related concepts

- [Tiered Storage for Logs](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Tiered_Storage_for_Logs.md>) *(Event Driven Architecture)*
- [Consuming Untrusted Streams](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Provisional_and_Final_Results/Consuming_Untrusted_Streams.md>) *(Streaming Architecture)*
- [End to End Exactly Once](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/End_to_End_Exactly_Once.md>) *(Big Data Processing)*

