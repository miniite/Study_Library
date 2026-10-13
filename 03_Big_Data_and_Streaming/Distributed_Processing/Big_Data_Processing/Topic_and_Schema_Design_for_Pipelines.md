---
id: "03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Topic_and_Schema_Design_for_Pipelines"
level: 4
type: "concept"
topic: "03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing"
tools: ["Apache_Kafka", "Apache_Spark", "Hadoop"]
---

# Topic and Schema Design for Pipelines

**L4 Advanced** · [↑ Big Data Processing](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Apache Kafka](<../../../TOOLS/Apache_Kafka/_index.md>)
- [Apache Spark](<../../../TOOLS/Apache_Spark/_index.md>)
- [Hadoop](<../../../TOOLS/Hadoop/_index.md>)

## Practice questions

- [Basic] [Why would you send different signal types like metrics logs and traces to separate Kafka topics?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Q-04623_Why_would_you_send_different_signal_types_like_metrics_logs_.md>)
- [Basic] [What could go wrong if two teams send logs with different field names into the same Kafka topic?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Q-04625_What_could_go_wrong_if_two_teams_send_logs_with_different_fi.md>)
- [Intermediate] [Why would you agree on a schema for events before producing them to a Kafka topic?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Q-04852_Why_would_you_agree_on_a_schema_for_events_before_producing_.md>)
- [Deep] [How would you evolve the schema of a telemetry topic without breaking existing Spark consumers?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Q-04685_How_would_you_evolve_the_schema_of_a_telemetry_topic_without.md>)
- [Deep] [You need to add three new fields to a telemetry schema without breaking 40 running Spark consumers; how do you roll that out safely?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Q-04705_You_need_to_add_three_new_fields_to_a_telemetry_schema_witho.md>)
- [Deep] [How would you structure Kafka topics and Spark jobs so that adding a new signal type (say traces) doesn't require touching the metrics or logs pipelines?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Q-04720_How_would_you_structure_Kafka_topics_and_Spark_jobs_so_that_.md>)
- [Deep] [How would you handle out-of-order CDC events (an update arriving before its insert) when materializing the current state of a table in Spark?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Q-04911_How_would_you_handle_out-of-order_CDC_events_an_update_arriv.md>)
- [Deep] [You want to switch a topic from JSON to Avro without downtime; walk me through a dual-write or dual-read migration and how you'd verify no records are lost.](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Q-04929_You_want_to_switch_a_topic_from_JSON_to_Avro_without_downtim.md>)

## Related concepts

- [Kafka and the Lakehouse](<../../../TOOLS/Apache_Kafka/Kafka_Core/Kafka_and_the_Lakehouse.md>) *(Kafka Core)*
- [Kafka Streams and Event Time](<../../../TOOLS/Apache_Kafka/Kafka_Core/Kafka_Streams_and_Event_Time/_index.md>) *(Kafka Core)*
- [Tiered Storage for Logs](<../../Streaming_and_Messaging/Event_Driven_Architecture/Tiered_Storage_for_Logs.md>) *(Event Driven Architecture)*

