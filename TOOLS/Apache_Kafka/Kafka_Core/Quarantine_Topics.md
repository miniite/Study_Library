---
id: "TOOLS/Apache_Kafka/Kafka_Core/Quarantine_Topics"
level: 4
type: "concept"
topic: "TOOLS/Apache_Kafka/Kafka_Core"
tools: ["Apache_Kafka"]
---

# Quarantine Topics

**L4 Advanced** · [↑ Kafka Core](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Apache Kafka](<../_index.md>)

## Practice questions

- [Basic] [How do you decide whether an event should carry the full record or just an ID that the pipeline looks up later?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-31009_How_do_you_decide_whether_an_event_should_carry_the_full_rec.md>)
- [Intermediate] [A malformed record from an upstream team keeps failing your parser. How would you keep the pipeline moving instead of stopping on it?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-31023_A_malformed_record_from_an_upstream_team_keeps_failing_your_.md>)
- [Intermediate] [How would you make sure the schema of events matches the columns your warehouse table expects, and catch a mismatch early?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-31056_How_would_you_make_sure_the_schema_of_events_matches_the_col.md>)
- [Intermediate] [Your Kafka Connect sink task keeps failing on one bad record and stalls the whole topic; how do you configure error tolerance and a DLQ so the load keeps moving?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-31142_Your_Kafka_Connect_sink_task_keeps_failing_on_one_bad_record.md>)
- [Deep] [What is Change Data Capture (CDC) and how is Kafka commonly used with Debezium?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-30579_What_is_Change_Data_Capture_CDC_and_how_is_Kafka_commonly_us.md>)
- [Deep] [How would you design the outbox table and CDC connector so your service never dual-writes and events are never lost even if the app crashes after DB commit?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-30772_How_would_you_design_the_outbox_table_and_CDC_connector_so_y.md>)
- [Deep] [Your CDC topic from Debezium sends before/after images; how do you build an upsert into an Iceberg or Delta table that correctly applies inserts, updates, and deletes in order?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-31103_Your_CDC_topic_from_Debezium_sends_before_after_images_how_d.md>)
- [Deep] [You have one bad record stuck at the head of a partition choking the warehouse loader; what options besides blindly skipping it preserve data-quality and replayability?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-31175_You_have_one_bad_record_stuck_at_the_head_of_a_partition_cho.md>)

## Related concepts

- [Topic and Schema Design for Pipelines](<../../../03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Topic_and_Schema_Design_for_Pipelines.md>) *(Big Data Processing)*
- [End to End Exactly Once](<../../../03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/End_to_End_Exactly_Once.md>) *(Big Data Processing)*
- [Consuming Untrusted Streams](<../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Provisional_and_Final_Results/Consuming_Untrusted_Streams.md>) *(Streaming Architecture)*

