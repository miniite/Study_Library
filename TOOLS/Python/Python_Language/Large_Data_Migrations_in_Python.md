---
id: "TOOLS/Python/Python_Language/Large_Data_Migrations_in_Python"
level: 4
type: "concept"
topic: "TOOLS/Python/Python_Language"
tools: ["Python", "Apache_Spark"]
---

# Large Data Migrations in Python

**L4 Advanced** · [↑ Python Language](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Python](<../_index.md>)
- [Apache Spark](<../../Apache_Spark/_index.md>)

## Practice questions

- [Basic] [What is the difference between batch processing and stream processing, and where would a Python batch job sit in a data platform versus a streaming consumer?](<../../../PRACTICE/TOOLS/Python/Python_Language/Q-37876_What_is_the_difference_between_batch_processing_and_stream_p.md>)
- [Intermediate] [How would you parallelize independent extraction tasks in Python (for example fetching many API endpoints) to speed up a pipeline, and what should you watch out for?](<../../../PRACTICE/TOOLS/Python/Python_Language/Q-37908_How_would_you_parallelize_independent_extraction_tasks_in_Py.md>)
- [Deep] [How would you implement chunked, resumable processing so a long ETL job can restart from a checkpoint after a crash without reprocessing everything?](<../../../PRACTICE/TOOLS/Python/Python_Language/Q-36787_How_would_you_implement_chunked_resumable_processing_so_a_lo.md>)
- [Deep] [Your Python pipeline uses pandas to compute rolling anomaly stats but chokes on 50k concurrent series. How would you decide between vectorized numpy, incremental online updates, or moving to a streaming engine?](<../../../PRACTICE/TOOLS/Python/Python_Language/Q-37784_Your_Python_pipeline_uses_pandas_to_compute_rolling_anomaly_.md>)
- [Deep] [What is the difference between tumbling, sliding, and session windows in stream processing, and how would you express a 5-minute tumbling aggregation in PySpark?](<../../../PRACTICE/TOOLS/Python/Python_Language/Q-37949_What_is_the_difference_between_tumbling_sliding_and_session_.md>)
- [Deep] [How would you run a data migration in Python over 50M rows in production without spiking replication lag or holding long locks, chunking and pacing the work?](<../../../PRACTICE/TOOLS/Python/Python_Language/Q-38012_How_would_you_run_a_data_migration_in_Python_over_50M_rows_i.md>)
- [Expert] [Explain when multiprocessing shared memory or a Manager beats serializing large arrays between processes, and the pickling cost you're trying to avoid.](<../../../PRACTICE/TOOLS/Python/Python_Language/Q-36752_Explain_when_multiprocessing_shared_memory_or_a_Manager_beat.md>)
- [Expert] [Explain watermarking in Spark Structured Streaming from Python: how does it bound state for late data, and what happens to events that arrive after the watermark?](<../../../PRACTICE/TOOLS/Python/Python_Language/Q-37948_Explain_watermarking_in_Spark_Structured_Streaming_from_Pyth.md>)

## Related concepts

- [Structured Streaming in PySpark](<../../Apache_Spark/PySpark/Structured_Streaming_in_PySpark.md>) *(PySpark)*
- [Stream Join Data Structures](<../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Stream_Join_Data_Structures.md>) *(Data Structures and Algorithms)*
- [Clickstream Counting Pipeline](<../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Clickstream_Counting_Pipeline/_index.md>) *(System Design Core)*

