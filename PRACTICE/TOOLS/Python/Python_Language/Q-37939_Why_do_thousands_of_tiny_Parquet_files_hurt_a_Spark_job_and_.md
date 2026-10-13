---
type: "practice"
difficulty: "Deep"
---

# Why do thousands of tiny Parquet files hurt a Spark job, and how would you write Python logic to compact them into right-sized files during your pipeline?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Pandas vs Polars vs PySpark](<../../../../TOOLS/Python/Python_Language/Pandas_vs_Polars_vs_PySpark.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand why thousands of tiny Parquet files can hurt a Spark job and how to address this issue using Python in a data engineering pipeline, we need to explore several key concepts related to file management, performance optimization, and best practices in big data processing.

The core idea is that having a large number of small files can lead to inefficiencies in both Spark's execution planning and I/O operations. Specifically, each small Parquet file requires its own metadata read and decompression, which can add overhead and slow down the job. Additionally, Spark’s partitioning and caching mechanisms may not perform optimally when dealing with a large number of small files.

To compact these tiny Parquet files into right-sized chunks during the pipeline, we need to write Python logic that aggregates smaller files based on certain criteria, such as file size or data content.

