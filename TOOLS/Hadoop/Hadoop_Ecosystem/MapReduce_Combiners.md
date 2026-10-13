---
id: "TOOLS/Hadoop/Hadoop_Ecosystem/MapReduce_Combiners"
level: 4
type: "concept"
topic: "TOOLS/Hadoop/Hadoop_Ecosystem"
tools: ["Hadoop"]
---

# MapReduce Combiners

**L4 Advanced** · [↑ Hadoop Ecosystem](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Hadoop](<../_index.md>)

## Practice questions

- [Basic] [What is MapReduce and how does it break a large data processing job into smaller pieces?](<../../../PRACTICE/TOOLS/Hadoop/Hadoop_Ecosystem/Q-29394_What_is_MapReduce_and_how_does_it_break_a_large_data_process.md>)
- [Basic] [What is data compression in Hadoop and why might you compress the output of a job?](<../../../PRACTICE/TOOLS/Hadoop/Hadoop_Ecosystem/Q-29429_What_is_data_compression_in_Hadoop_and_why_might_you_compres.md>)
- [Intermediate] [How does the number of mappers get decided for a MapReduce job?](<../../../PRACTICE/TOOLS/Hadoop/Hadoop_Ecosystem/Q-29407_How_does_the_number_of_mappers_get_decided_for_a_MapReduce_j.md>)
- [Intermediate] [Explain a real situation where you tuned mapreduce.job.reduces and what signals told you the value was wrong. What happens if it is set too high versus too low?](<../../../PRACTICE/TOOLS/Hadoop/Hadoop_Ecosystem/Q-29486_Explain_a_real_situation_where_you_tuned_mapreduce_job_reduc.md>)
- [Deep] [How do you decide or configure the number of reducers for a MapReduce job and what happens if you set it to zero?](<../../../PRACTICE/TOOLS/Hadoop/Hadoop_Ecosystem/Q-29408_How_do_you_decide_or_configure_the_number_of_reducers_for_a_.md>)
- [Deep] [What is a map-side join versus a reduce-side join in Hadoop and when would you use each?](<../../../PRACTICE/TOOLS/Hadoop/Hadoop_Ecosystem/Q-29446_What_is_a_map-side_join_versus_a_reduce-side_join_in_Hadoop_.md>)
- [Deep] [How do you decide the right compression codec (Snappy, Gzip, LZO, Zstd) for intermediate map output versus final table storage, and why does splittability matter?](<../../../PRACTICE/TOOLS/Hadoop/Hadoop_Ecosystem/Q-29502_How_do_you_decide_the_right_compression_codec_Snappy_Gzip_LZ.md>)
- [Deep] [Explain how you would implement a custom partitioner to fix skew where a few hot keys dominate one reducer.](<../../../PRACTICE/TOOLS/Hadoop/Hadoop_Ecosystem/Q-29543_Explain_how_you_would_implement_a_custom_partitioner_to_fix_.md>)

## Related concepts

- [Distributed Joins in MPP Systems](<../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Joins_in_MPP_Systems/_index.md>) *(Distributed Systems Core)*
- [Shuffle Partition Tuning](<../../../03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Shuffle_Partition_Tuning.md>) *(Big Data Processing)*
- [Spark Skew and Spill](<../../Apache_Spark/Spark_Optimization/Spark_Skew_and_Spill.md>) *(Spark Optimization)*

## References

- *Building Medallion Architectures* — Piethein Strengholt — The Evolution of Data Architecture › MapReduce
- *The Enterprise Big Data Lake* — Alex Gorelik — Introduction to Big Data and Data Science › How Processing and Storage Interact in a MapReduce Job
- *Hadoop: The Definitive Guide* — Tom White — Meet Hadoop › Querying All Your Data
- *Hadoop: The Definitive Guide* — Tom White — Meet Hadoop › Comparison with Other Systems
- *Hadoop: The Definitive Guide* — Tom White — MapReduce › Chapter 2. MapReduce
- *Hadoop: The Definitive Guide* — Tom White — MapReduce › Analyzing the Data with Hadoop
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Prerequisite Knowledge for Aspiring Data Engineers › MapReduce
- *Designing Data-Intensive Applications* — Martin Kleppmann — Data Models and Query Languages › MapReduce Querying
- *Designing Data-Intensive Applications* — Martin Kleppmann — The Unix Philosophy › MapReduce and Distributed Filesystems
- *Designing Data-Intensive Applications* — Martin Kleppmann — MapReduce Job Execution
- *Designing Data-Intensive Applications* — Martin Kleppmann — MapReduce Job Execution › Distributed execution of MapReduce
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Modeling Streaming Data › MapReduce
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Modeling Streaming Data › After MapReduce

