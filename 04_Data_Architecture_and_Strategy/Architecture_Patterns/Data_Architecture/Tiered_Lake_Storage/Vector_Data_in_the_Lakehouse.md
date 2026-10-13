---
id: "04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Vector_Data_in_the_Lakehouse"
level: 5
type: "deep-dive"
topic: "04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture"
tools: ["AWS"]
---

# Vector Data in the Lakehouse

**L5 Expert** · [↑ Data Architecture](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [AWS](<../../../../TOOLS/AWS/_index.md>)

## Practice questions

- [Intermediate] [When would you choose a column-oriented format like Parquet over row-oriented Avro for your training data, and when the reverse for streaming ingestion?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10703_When_would_you_choose_a_column-oriented_format_like_Parquet_.md>)
- [Intermediate] [Explain when you'd choose Parquet versus Avro for a given stage of your pipeline. Where does row-oriented Avro win and where does columnar Parquet win?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-11089_Explain_when_you_d_choose_Parquet_versus_Avro_for_a_given_st.md>)
- [Intermediate] [You're deciding whether embeddings for a new search feature belong in a lake table with a vector index or a dedicated vector DB. As a DE, how do you keep them in lineage and governed either way?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-11128_You_re_deciding_whether_embeddings_for_a_new_search_feature_.md>)
- [Intermediate] [Explain how you'd model geospatial data (locations, geofences) across your lake and warehouse for a delivery product, and where forcing it into a generic columnar store falls apart.](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-11131_Explain_how_you_d_model_geospatial_data_locations_geofences_.md>)
- [Deep] [Your team is debating storing embeddings in the lake versus a dedicated vector store. How do you reason about that split for a RAG pipeline?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10676_Your_team_is_debating_storing_embeddings_in_the_lake_versus_.md>)
- [Deep] [Explain how deletion vectors work for row-level deletes in a lakehouse and the read-time cost of letting them accumulate. What compaction cadence keeps queries fast?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-11117_Explain_how_deletion_vectors_work_for_row-level_deletes_in_a.md>)
- [Deep] [How do you architect vector storage and retrieval so it's a governed part of your data platform rather than a rogue side database? When do embeddings belong in a lakehouse column with a vector index versus a dedicated vector DB, and how do you keep them in lineage?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10960_How_do_you_architect_vector_storage_and_retrieval_so_it_s_a_.md>)
- [Deep] [How do you architect a data platform for a genomics/research org where individual datasets are terabyte-scale files (BAM/VCF) that don't fit a columnar table model? Where does object-storage-plus-metadata-index beat a warehouse, and how do you still provide catalog, lineage, and governance over opaque files?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-11021_How_do_you_architect_a_data_platform_for_a_genomics_research.md>)

## Related concepts

- [Choosing a Database for the Workload](<../../../../01_Databases_and_SQL/Database_Strategy/Database_Selection/Choosing_a_Database_for_the_Workload.md>) *(Database Selection)*
- [Telemetry Lake File Layout](<../../Observability_Data_Lake/Telemetry_Lake_File_Layout.md>) *(Observability Data Lake)*
- [Index Strategy and Governance at Scale](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Index_Tuning_in_Practice/Index_Strategy_and_Governance_at_Scale.md>) *(Indexing)*

## References

- *97 Things Every Data Engineer Should Know* — Tobias Macey (ed.) — Embrace the Data Lake Architecture

