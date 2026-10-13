---
id: "TOOLS/Apache_Kafka/Kafka_Core/Topic_Design_for_Re-sync"
level: 4
type: "concept"
topic: "TOOLS/Apache_Kafka/Kafka_Core"
tools: ["Apache_Kafka"]
---

# Topic Design for Re-sync

**L4 Advanced** · [↑ Kafka Core](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Apache Kafka](<../_index.md>)

## Practice questions

- [Basic] [What is a Kafka topic and how is it different from a queue in traditional messaging systems?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-30512_What_is_a_Kafka_topic_and_how_is_it_different_from_a_queue_i.md>)
- [Intermediate] [What does kafka-topics.sh --describe tell you, and how do you read the Isr and Replicas fields to spot an under-replicated partition?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-30657_What_does_kafka-topics_sh_--describe_tell_you_and_how_do_you.md>)
- [Intermediate] [Your team debates one big topic with a type field versus many small topics for different event types. What are the trade-offs for ingestion?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-31042_Your_team_debates_one_big_topic_with_a_type_field_versus_man.md>)
- [Deep] [How would you design a system to ensure events for the same user are processed in order?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-30580_How_would_you_design_a_system_to_ensure_events_for_the_same_.md>)
- [Deep] [How would you grant a service write access to only one topic prefix using ACLs, and what resource patterns exist?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-30697_How_would_you_grant_a_service_write_access_to_only_one_topic.md>)
- [Deep] [How would you design at-least-once delivery from an alert topic into PagerDuty so no critical page is ever lost, even across a total consumer restart?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-30928_How_would_you_design_at-least-once_delivery_from_an_alert_to.md>)
- [Deep] [How would you design topic-level access control so that a compromised producer credential can write to at most one topic and cannot read anything?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-30509_How_would_you_design_topic-level_access_control_so_that_a_co.md>)
- [Expert] [How would you design topics for a multi-tenant system: topic-per-tenant vs key-per-tenant, and the trade-offs?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-30674_How_would_you_design_topics_for_a_multi-tenant_system_topic-.md>)

## Related concepts

- [Partition Keys](<../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Partition_Keys.md>) *(Event Driven Architecture)*
- [Topic and Schema Design for Pipelines](<../../../03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Topic_and_Schema_Design_for_Pipelines.md>) *(Big Data Processing)*
- [Tiered Storage for Logs](<../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Tiered_Storage_for_Logs.md>) *(Event Driven Architecture)*

