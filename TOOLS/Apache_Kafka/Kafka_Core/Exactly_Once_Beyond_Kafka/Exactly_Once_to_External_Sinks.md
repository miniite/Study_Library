---
id: "TOOLS/Apache_Kafka/Kafka_Core/Exactly_Once_to_External_Sinks"
level: 5
type: "deep-dive"
topic: "TOOLS/Apache_Kafka/Kafka_Core"
tools: ["Apache_Kafka"]
---

# Exactly Once to External Sinks

**L5 Expert** · [↑ Kafka Core](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Apache Kafka](<../../_index.md>)

## Practice questions

- [Deep] [How would you design a topic and consumer so that reprocessing after a bug fix is idempotent and produces the exact same downstream state as if the bug never happened?](<../../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-30488_How_would_you_design_a_topic_and_consumer_so_that_reprocessi.md>)
- [Expert] [How would you architect exactly-once delivery from a Kafka topic into Elasticsearch given ES has no transactions? Discuss versioning and optimistic concurrency.](<../../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-30752_How_would_you_architect_exactly-once_delivery_from_a_Kafka_t.md>)
- [Expert] [Design a fleet-wide standard for exactly-once-effective delivery into heterogeneous sinks (Postgres Elasticsearch Redis S3) each lacking Kafka-native transactions. What common pattern (idempotent upsert, versioning, offset-in-sink) would you mandate, and where must each sink deviate?](<../../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-30882_Design_a_fleet-wide_standard_for_exactly-once-effective_deli.md>)

## Related concepts

- [End to End Exactly Once](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/End_to_End_Exactly_Once.md>) *(Big Data Processing)*
- [Consuming Untrusted Streams](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Provisional_and_Final_Results/Consuming_Untrusted_Streams.md>) *(Streaming Architecture)*
- [Tiered Storage for Logs](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Tiered_Storage_for_Logs.md>) *(Event Driven Architecture)*

