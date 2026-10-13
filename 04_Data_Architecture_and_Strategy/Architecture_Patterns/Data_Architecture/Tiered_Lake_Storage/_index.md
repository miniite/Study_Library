---
id: "04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage"
level: 4
type: "concept"
topic: "04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture"
tools: ["AWS", "Databricks", "Apache_Spark", "Hadoop", "Apache_Flink"]
---

# Tiered Lake Storage

**L4 Advanced** · [↑ Data Architecture](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Go deeper

- [End to End No Data Loss](<End_to_End_No_Data_Loss.md>)
- [Reproducible Training Data Versions](<Reproducible_Training_Data_Versions.md>)
- [Small Files and Compaction](<Small_Files_and_Compaction.md>)
- [Vector Data in the Lakehouse](<Vector_Data_in_the_Lakehouse.md>)
- [Streaming-first Data Architecture](<Streaming-first_Data_Architecture.md>)
- [Rollups vs On-demand Queries](<Rollups_vs_On-demand_Queries.md>)
- [Real-time OLAP Serving Layer](<Real-time_OLAP_Serving_Layer.md>)

## Implementations (tools)

- [AWS](<../../../../TOOLS/AWS/_index.md>)
- [Databricks](<../../../../TOOLS/Databricks/_index.md>)
- [Apache Spark](<../../../../TOOLS/Apache_Spark/_index.md>)
- [Hadoop](<../../../../TOOLS/Hadoop/_index.md>)
- [Apache Flink](<../../../../TOOLS/Apache_Flink/_index.md>)

## Practice questions

- [Basic] [Explain the practical difference between a data lake and a lakehouse to a backend engineer who thinks 'we already dump JSON in S3, so we have a lake.' What are we actually missing without a table format?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-11049_Explain_the_practical_difference_between_a_data_lake_and_a_l.md>)
- [Intermediate] [Explain how tiered storage (hot/warm/cold) on the lake affects your ability to run a training job over old data, and what surprises you on cost or latency.](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10709_Explain_how_tiered_storage_hot_warm_cold_on_the_lake_affects.md>)
- [Intermediate] [Explain what time-travel gives you in Iceberg or Delta and a real situation where it saved you. What does keeping those snapshots cost in storage?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-11106_Explain_what_time-travel_gives_you_in_Iceberg_or_Delta_and_a.md>)
- [Deep] [Your model needs a snapshot of features exactly as they were at prediction time last month for a backtest. How does time-travel in Delta or Iceberg help, and what's it costing you in storage?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10655_Your_model_needs_a_snapshot_of_features_exactly_as_they_were.md>)
- [Deep] [Your lake stores slowly changing dimensions (like a user's plan tier). How do you pick the right feature value for a training example from a given date?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10687_Your_lake_stores_slowly_changing_dimensions_like_a_user_s_pl.md>)
- [Deep] [You need to compute a rolling 30-day feature per user over billions of events. How does the lake's partitioning and file layout make or break this job?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10734_You_need_to_compute_a_rolling_30-day_feature_per_user_over_b.md>)
- [Deep] [Explain the difference between partitioning and bucketing in a table and a case where you'd bucket on a join key. What's the trap of over-partitioning a wide fact table?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-11125_Explain_the_difference_between_partitioning_and_bucketing_in.md>)
- [Deep] [Your ML teams pull training data from a warehouse that only retains 90 days, but seasonal models need 3 years. Architect the retention/tiering split between warehouse and lake so long-horizon training is possible without paying warehouse rates for cold data.](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10824_Your_ML_teams_pull_training_data_from_a_warehouse_that_only_.md>)

## Related concepts

- [Lakehouse Fundamentals](<../../../../02_Data_Engineering/Storage_and_Lakehouse/Lakehouse/Lakehouse_Fundamentals.md>) *(Lakehouse)*
- [File and Table Formats for Features](<../../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/File_and_Table_Formats_for_Features.md>) *(Pipeline Orchestration)*

## References

- *Practical Lakehouse Architecture* — Gaurav Ashok Thalpati — Storage The Heart of the Lakehouse

