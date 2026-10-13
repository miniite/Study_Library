---
type: "practice"
difficulty: "Basic"
---

# What is a primary key and why does a pipeline need to know it to avoid duplicate rows?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Exactly Once Spark Writes](<../../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Exactly_Once_Spark_Writes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: A primary key uniquely identifies each row in a database table, ensuring data integrity. In a data pipeline like Apache Airflow or Spark, knowing the primary key is crucial to avoid processing duplicate rows multiple times, which can lead to inefficiencies and incorrect results.

#### Triage
The question probes your understanding of primary keys and their role in maintaining data consistency within a pipeline. We need to assume that the pipeline processes large volumes of data from various sources and aims to maintain uniqueness in processed datasets.

#### First-principles solution

Understanding Primary Keys: A primary key is an attribute or combination of attributes uniquely identifying each record in a database table. It ensures that no two rows have the same value, making it essential for maintaining data integrity.
Pipeline Considerations:
Avoiding Duplicates: When processing large datasets, ensuring uniqueness prevents redundant operations and maintains consistency.
Efficiency: Identifying duplicates early can significantly improve performance by reducing unnecessary computations.
Correctness: Ensuring no duplicates means the processed data accurately reflects the intended state.

To incorporate this understanding into a pipeline:

Airflow: Use sensors or operators that check for existing records based on primary keys before inserting new ones. For example, using BigQueryCheckOperator to verify if a record exists in BigQuery.
Spark: Utilize RDD transformations like filterNot or DataFrame operations such as dropDuplicates() to handle duplicates.

#### Closing summary
and common mistakes/follow-ups: In data pipelines, understanding the primary key is fundamental for ensuring data integrity. Mistakes often arise from not properly handling duplicate rows, leading to inefficiencies and incorrect results. Common follow-ups might include discussing how different database management systems (DBMS) like MySQL or PostgreSQL handle primary keys differently or exploring more advanced techniques like using composite primary keys in Spark.

