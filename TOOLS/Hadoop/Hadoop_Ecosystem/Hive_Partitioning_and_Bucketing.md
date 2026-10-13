---
id: "TOOLS/Hadoop/Hadoop_Ecosystem/Hive_Partitioning_and_Bucketing"
level: 4
type: "concept"
topic: "TOOLS/Hadoop/Hadoop_Ecosystem"
tools: ["Hadoop", "Apache_Spark"]
---

# Hive Partitioning and Bucketing

**L4 Advanced** · [↑ Hadoop Ecosystem](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Hadoop](<../_index.md>)
- [Apache Spark](<../../Apache_Spark/_index.md>)

## Practice questions

- [Basic] [What is Apache Hive and how does it make working with Hadoop easier for someone who knows SQL?](<../../../PRACTICE/TOOLS/Hadoop/Hadoop_Ecosystem/Q-29409_What_is_Apache_Hive_and_how_does_it_make_working_with_Hadoop.md>)
- [Intermediate] [What are user defined functions (UDFs) in Hive and when would you write one?](<../../../PRACTICE/TOOLS/Hadoop/Hadoop_Ecosystem/Q-29451_What_are_user_defined_functions_UDFs_in_Hive_and_when_would_.md>)
- [Deep] [What is the difference between the ORDER BY SORT BY DISTRIBUTE BY and CLUSTER BY clauses in Hive?](<../../../PRACTICE/TOOLS/Hadoop/Hadoop_Ecosystem/Q-29450_What_is_the_difference_between_the_ORDER_BY_SORT_BY_DISTRIBU.md>)
- [Deep] [Explain how you would design daily partitioning plus bucketing for a large clickstream Hive table, and how you would choose the number of buckets.](<../../../PRACTICE/TOOLS/Hadoop/Hadoop_Ecosystem/Q-29491_Explain_how_you_would_design_daily_partitioning_plus_bucketi.md>)
- [Deep] [A daily Hive aggregation produces wrong totals only on days with DST changes or timezone-crossing data. How would you trace this to partition keys and timestamp handling?](<../../../PRACTICE/TOOLS/Hadoop/Hadoop_Ecosystem/Q-29521_A_daily_Hive_aggregation_produces_wrong_totals_only_on_days_.md>)
- [Deep] [A Hive UDF you wrote is slow because it is called per-row on billions of rows. How would you optimise or replace it with a native/built-in approach?](<../../../PRACTICE/TOOLS/Hadoop/Hadoop_Ecosystem/Q-29547_A_Hive_UDF_you_wrote_is_slow_because_it_is_called_per-row_on.md>)
- [Deep] [A Hive query with many small ORC stripes reads slowly. How do stripe size, row-group size, and file size interact with read performance?](<../../../PRACTICE/TOOLS/Hadoop/Hadoop_Ecosystem/Q-29566_A_Hive_query_with_many_small_ORC_stripes_reads_slowly_How_do.md>)
- [Expert] [Your ORC-based Hive tables are still slow on selective filters. How do you use predicate pushdown, bloom filters, and stripe/row-group sizing to speed them up?](<../../../PRACTICE/TOOLS/Hadoop/Hadoop_Ecosystem/Q-29503_Your_ORC-based_Hive_tables_are_still_slow_on_selective_filte.md>)

## Related concepts

- [Partitioning and Bucketing Design](<../../../02_Data_Engineering/Storage_and_Lakehouse/Partitioning_and_Bucketing/Partitioning_and_Bucketing_Design.md>) *(Partitioning and Bucketing)*
- [Partitioning and Data Skipping in Lakes](<../../../01_Databases_and_SQL/Query_Performance/Indexing/Partitioning_and_Data_Skipping_in_Lakes.md>) *(Indexing)*
- [Lakehouse Fundamentals](<../../../02_Data_Engineering/Storage_and_Lakehouse/Lakehouse/Lakehouse_Fundamentals.md>) *(Lakehouse)*

## References

- *Hadoop: The Definitive Guide* — Tom White — Sqoop › Imported Data and Hive
- *Hadoop: The Definitive Guide* — Tom White — Sqoop › Note
- *Hadoop: The Definitive Guide* — Tom White — Comparison with Databases › Using Hive tables with HCatalog
- *Hadoop: The Definitive Guide* — Tom White — Hive › Installing Hive
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Prerequisite Knowledge for Aspiring Data Engineers › Hive
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Prerequisite Knowledge for Aspiring Data Engineers › Presto
- *Practical Lakehouse Architecture* — Gaurav Ashok Thalpati — Storage The Heart of the Lakehouse › Key features
- *Practical Lakehouse Architecture* — Gaurav Ashok Thalpati — Storage The Heart of the Lakehouse › Hive
- *Optimizing Databricks Workloads* — Anirudh Kala, Anshul Bhatnagar, Sarthak Sarbahi — Big Data Analytics › Understanding Hive partitions
- *Building Medallion Architectures* — Piethein Strengholt — The Evolution of Data Architecture › External and internal tables
- *Building Medallion Architectures* — Piethein Strengholt — The Evolution of Data Architecture › Hive Metastore
- *Building Medallion Architectures* — Piethein Strengholt — H
- *Building Medallion Architectures* — Piethein Strengholt — H › M
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Diving into the Delta Lake Ecosystem › Running the Hive Metastore
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Metadata Management, Data Flow, and Lineage › Why the Metastore Matters
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — T

