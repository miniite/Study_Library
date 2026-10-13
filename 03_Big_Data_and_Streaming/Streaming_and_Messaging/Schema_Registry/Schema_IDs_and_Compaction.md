---
id: "03_Big_Data_and_Streaming/Streaming_and_Messaging/Schema_Registry/Schema_IDs_and_Compaction"
level: 4
type: "concept"
topic: "03_Big_Data_and_Streaming/Streaming_and_Messaging/Schema_Registry"
tools: ["Apache_Kafka", "AWS"]
---

# Schema IDs and Compaction

**L4 Advanced** · [↑ Schema Registry](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Apache Kafka](<../../../TOOLS/Apache_Kafka/_index.md>)
- [AWS](<../../../TOOLS/AWS/_index.md>)

## Practice questions

- [Intermediate] [How does schema-id caching in the producer/consumer client work and what failure did you see the first time your Schema Registry cluster was unreachable at startup vs mid-stream?](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Schema_Registry/Q-39057_How_does_schema-id_caching_in_the_producer_consumer_client_w.md>)
- [Intermediate] [When a source table adds a column mid-stream in a Debezium+Kafka+Snowflake sink pipeline what exactly must evolve at the registry the connector and the warehouse — trace the chain.](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Schema_Registry/Q-39073_When_a_source_table_adds_a_column_mid-stream_in_a_Debezium_K.md>)
- [Intermediate] [How do you version and store your .avsc/.proto schema definitions in source control and keep them the source of truth versus the registry?](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Schema_Registry/Q-39118_How_do_you_version_and_store_your_avsc_proto_schema_definiti.md>)
- [Deep] [Compare Avro Protobuf and JSON Schema as serialization formats behind a registry — which would you standardize on for a new event backbone and what trade-offs drove it?](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Schema_Registry/Q-39044_Compare_Avro_Protobuf_and_JSON_Schema_as_serialization_forma.md>)
- [Deep] [Explain how the _schemas Kafka topic underpins Confluent Schema Registry and what happens to registrations if that topic loses a partition leader.](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Schema_Registry/Q-39063_Explain_how_the_schemas_Kafka_topic_underpins_Confluent_Sche.md>)
- [Deep] [A CDC pipeline (Debezium) emits schema changes when the source DDL changes. How does Debezium encode schema evolution and how do you keep sink connectors from breaking?](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Schema_Registry/Q-39072_A_CDC_pipeline_Debezium_emits_schema_changes_when_the_source.md>)
- [Deep] [BigQuery supports adding columns and relaxing REQUIRED to NULLABLE but not much else. How do you design ingestion so schema drift doesn't force full reloads?](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Schema_Registry/Q-39108_BigQuery_supports_adding_columns_and_relaxing_REQUIRED_to_NU.md>)
- [Deep] [Explain the Confluent wire-format magic byte and 4-byte schema id — what breaks if a consumer tries to deserialize a message that doesn't have this framing?](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Schema_Registry/Q-39116_Explain_the_Confluent_wire-format_magic_byte_and_4-byte_sche.md>)

## Related concepts

- [Schema Formats and Registry](<../../../TOOLS/Apache_Kafka/Kafka_Core/Schema_Formats_and_Registry.md>) *(Kafka Core)*
- [Schema Changes Under Long Transactions](<../../../04_Data_Architecture_and_Strategy/Platform_Strategy/Schema_Evolution/Schema_Changes_Under_Long_Transactions.md>) *(Schema Evolution)*
- [Avro Schema Evolution](<../../../02_Data_Engineering/Storage_and_Lakehouse/File_Formats/Avro_Schema_Evolution.md>) *(File Formats)*

## References

- *Delta Lake: Up and Running* — Bennie Haelen, Dan Davis — Schema Handling
- *Unifying Business, Data, and Code* — Ron Itelman, Juan Cruz Viotti — A Universal Language for Data
- *Unifying Business, Data, and Code* — Ron Itelman, Juan Cruz Viotti — Deploying a JSON Schema Registry
- *Unifying Business, Data, and Code* — Ron Itelman, Juan Cruz Viotti — Extending JSON Schema

