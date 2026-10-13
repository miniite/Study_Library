---
id: "03_Big_Data_and_Streaming/Streaming_and_Messaging/Schema_Registry/Protobuf_vs_Avro_Evolution"
level: 4
type: "concept"
topic: "03_Big_Data_and_Streaming/Streaming_and_Messaging/Schema_Registry"
tools: ["Databricks"]
---

# Protobuf vs Avro Evolution

**L4 Advanced** · [↑ Schema Registry](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Databricks](<../../../TOOLS/Databricks/_index.md>)

## Practice questions

- [Intermediate] [A Parquet file has type INT32 for a column that later files store as INT64. How does your query engine handle that on read and how do you prevent the mismatch at the pipeline level?](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Schema_Registry/Q-39045_A_Parquet_file_has_type_INT32_for_a_column_that_later_files_.md>)
- [Intermediate] [dbt model contracts can enforce column types at build. How would you wire that so a breaking model change fails CI before it hits downstream BI?](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Schema_Registry/Q-39086_dbt_model_contracts_can_enforce_column_types_at_build_How_wo.md>)
- [Deep] [Explain Avro schema resolution rules for promoting an int to a long or a float to a double — which direction is safe and why does the reader schema govern it?](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Schema_Registry/Q-39046_Explain_Avro_schema_resolution_rules_for_promoting_an_int_to.md>)
- [Deep] [What is column-mapping / rename support in Delta Lake and why does it change what counts as a breaking change downstream?](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Schema_Registry/Q-39056_What_is_column-mapping_rename_support_in_Delta_Lake_and_why_.md>)
- [Deep] [What is a logical type in Avro (e.g. decimal timestamp-millis uuid) and how do you evolve a field to a logical type without breaking older readers?](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Schema_Registry/Q-39094_What_is_a_logical_type_in_Avro_e_g_decimal_timestamp-millis_.md>)
- [Deep] [In Protobuf why is reusing a deleted field number dangerous and what does the reserved keyword do for evolution safety?](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Schema_Registry/Q-39097_In_Protobuf_why_is_reusing_a_deleted_field_number_dangerous_.md>)
- [Deep] [How would you evolve units of a numeric field (e.g. cents to dollars, ms to seconds) safely given the type doesn't change?](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Schema_Registry/Q-39134_How_would_you_evolve_units_of_a_numeric_field_e_g_cents_to_d.md>)
- [Expert] [Compare how Protobuf and Avro treat a field that's present in the writer but absent in the reader — which drops it silently and which errors?](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Schema_Registry/Q-39098_Compare_how_Protobuf_and_Avro_treat_a_field_that_s_present_i.md>)

## Related concepts

- [Schema Changes Under Long Transactions](<../../../04_Data_Architecture_and_Strategy/Platform_Strategy/Schema_Evolution/Schema_Changes_Under_Long_Transactions.md>) *(Schema Evolution)*
- [Avro Schema Evolution](<../../../02_Data_Engineering/Storage_and_Lakehouse/File_Formats/Avro_Schema_Evolution.md>) *(File Formats)*
- [Schema Evolution in Lakehouse Tables](<../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Schema_Evolution_in_Lakehouse_Tables.md>) *(SQL Fundamentals)*

## References

- *Designing Data-Intensive Applications* — Martin Kleppmann — Encoding and Evolution

