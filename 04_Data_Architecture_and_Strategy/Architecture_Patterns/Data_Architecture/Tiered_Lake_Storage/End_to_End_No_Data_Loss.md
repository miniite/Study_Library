---
id: "04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/End_to_End_No_Data_Loss"
level: 5
type: "deep-dive"
topic: "04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture"
tools: ["Apache_Kafka"]
---

# End to End No Data Loss

**L5 Expert** · [↑ Data Architecture](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Apache Kafka](<../../../../TOOLS/Apache_Kafka/_index.md>)

## Practice questions

- [Intermediate] [Your training pipeline reads Parquet from the lake but a colleague reads the same table via the warehouse and gets different row counts. How would you debug this discrepancy between the lake and the warehouse copy?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10645_Your_training_pipeline_reads_Parquet_from_the_lake_but_a_col.md>)
- [Intermediate] [Your daily pipeline sometimes runs on a partition that only half-landed and produces a low count. How do you make the job detect completeness and refuse to run on incomplete data?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-11078_Your_daily_pipeline_sometimes_runs_on_a_partition_that_only_.md>)
- [Intermediate] [Explain how you'd generate an access audit trail so 'who queried this sensitive table when' is answerable. Where do per-engine native audit logs leave gaps an auditor would find?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-11137_Explain_how_you_d_generate_an_access_audit_trail_so_who_quer.md>)
- [Deep] [Explain the difference between transaction-time and valid-time when your model needs to know 'what was true as of the prediction date' versus 'what we recorded later'.](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10686_Explain_the_difference_between_transaction-time_and_valid-ti.md>)
- [Deep] [A late-arriving batch of yesterday's events showed up this morning and your daily aggregate is now wrong. How do you design the pipeline so it handles late data instead of silently undercounting?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-11064_A_late-arriving_batch_of_yesterday_s_events_showed_up_this_m.md>)
- [Deep] [Your Spark job that reads a partitioned table works on a sample in dev but times out on the full table in prod. What assumptions about data layout or skew likely broke between the two?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-11099_Your_Spark_job_that_reads_a_partitioned_table_works_on_a_sam.md>)
- [Deep] [Your streaming enrichment job joins every event to a product-catalog reference table. Where do you cache the reference and what happens to in-flight events when the catalog updates mid-stream?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-11122_Your_streaming_enrichment_job_joins_every_event_to_a_product.md>)
- [Deep] [You inherit a warehouse where SCD Type 2 was implemented inconsistently across 200 dimensions (some Type 1, some Type 2, some broken), corrupting historical accuracy. As principal, how do you audit, standardize, and backfill history where recoverable, and where do you accept that history is permanently lost?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-11037_You_inherit_a_warehouse_where_SCD_Type_2_was_implemented_inc.md>)

## Related concepts

- [Lakehouse Fundamentals](<../../../../02_Data_Engineering/Storage_and_Lakehouse/Lakehouse/Lakehouse_Fundamentals.md>) *(Lakehouse)*
- [Unified Batch and Streaming Semantics](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream/Batch_and_Stream_Reconciliation/Unified_Batch_and_Streaming_Semantics.md>) *(Batch vs Stream)*

