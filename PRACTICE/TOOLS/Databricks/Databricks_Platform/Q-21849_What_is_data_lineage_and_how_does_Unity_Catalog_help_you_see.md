---
type: "practice"
difficulty: "Intermediate"
---

# What is data lineage and how does Unity Catalog help you see where a table's data came from?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Databricks Ingestion](<../../../../TOOLS/Databricks/Databricks_Platform/Databricks_Ingestion.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: Data lineage in Databricks helps track the flow of data across transformations, while Unity Catalog enhances this by providing a centralized view of metadata and dependencies. Understanding both concepts is crucial for managing and auditing data pipelines.

#### Triage
The question asks you to explain two key aspects of Databricks—data lineage and Unity Catalog—and how they work together. The interviewer is probing your understanding of the mechanisms that ensure data integrity and traceability within a Data Warehouse environment.

#### Assumptions

You are familiar with basic concepts in data engineering, such as transformations, pipelines, and metadata management.
You have some experience or knowledge about Databricks and its components.

#### First-principles Solution

Data Lineage: Data lineage is a critical concept that tracks the transformation history of a dataset through various stages. In the context of Databricks, data lineage helps in understanding how raw data evolves into processed data as it moves through different transformations and operations within the data pipeline.

Imagine you have a raw dataset stored in a CSV file. This dataset undergoes several transformations to clean and enrich it before being loaded into an analytics database. Data lineage provides a visual or detailed trace of each step, making it easier to audit and troubleshoot issues when something goes wrong.

Unity Catalog: Unity Catalog is a component introduced by Databricks that helps manage metadata across multiple data sources and formats in a centralized way. It supports various file systems like HDFS, S3, and ADLS, as well as traditional databases. Unity Catalog not only simplifies the management of datasets but also integrates with Delta Lake, another feature offered by Databricks for ACID transactions.

Unity Catalog's role in data lineage is to provide a comprehensive view of where each table's data originated from and how it has been transformed over time. By integrating with metadata, Unity Catalog can show the entire history of transformations that have affected a particular dataset.

How They Work Together: When you use Unity Catalog along with Databricks notebooks and Delta Lake, you get a powerful tool for managing and auditing your data pipelines. For example, if a query fails due to a missing or corrupted file, Unity Catalog can help identify the source of the issue by showing the lineage of that particular dataset.

Here's an illustrative scenario:

You have two tables, raw_data and clean_data, in Delta Lake.
raw_data is stored in S3 and ingested into Databricks using Unity Catalog.
A Databricks notebook processes raw_data to generate clean_data.
Unity Catalog records the lineage of this transformation.

If clean_data is ever found to be corrupt, you can use Unity Catalog's metadata to trace back to raw_data, ensuring that any issues are addressed at their source.

#### Closing Summary
Data lineage and Unity Catalog together provide a robust framework for managing data in Databricks. Lineage ensures traceability, while Unity Catalog centralizes metadata management. This combination is essential for maintaining data integrity and facilitating effective data governance.

#### Common Mistakes/Follow-Ups

Misunderstanding the role of Delta Lake: It's important to recognize that Delta Lake provides ACID transactions and snapshot history, which complements Unity Catalog’s metadata management.
Overlooking the importance of real-time lineage: In scenarios where data is frequently updated, understanding the exact lineage can be more complex. Discussing how you would handle such cases could be beneficial.

**Comparison:**

```
Table Name	Location	Type
raw_data	/path/to/raw_data	Delta Lake
clean_data	/path/to/clean_data	Delta Lake
Transformation Steps
Read from S3 using Unity Catalog
Apply transformations
Write to Delta Lake
```

**Code:**

```python
# Example Databricks notebook snippet for processing raw_data to clean_data using Delta Lake
from delta import *
spark.sql("CREATE TABLE IF NOT EXISTS raw_data USING DELTA LOCATION '/path/to/raw_data'")
spark.sql("CREATE TABLE IF NOT EXISTS clean_data USING DELTA LOCATION '/path/to/clean_data'")

# Processing logic in a Databricks notebook
spark.read.format("delta").load("/path/to/raw_data") \
    .write.format("delta").saveAsTable("clean_data")
```

**Example input / output:**

```text
Input:
- raw_data stored in S3

Output:
- clean_data created and processed using Databricks notebook, recorded with Unity Catalog for lineage tracking.
```

**Data:**

```json
{
  "tables": [
    {
      "name": "raw_data",
      "location": "/path/to/raw_data",
      "type": "Delta Lake"
    },
    {
      "name": "clean_data",
      "location": "/path/to/clean_data",
      "type": "Delta Lake"
    }
  ]
}
```

**Flow:**

1. Raw Data in S3
2. Unity Catalog
3. Read into DatabricksNotebook
4. Transform Data
5. Write Clean Data to Delta Lake
6. Record Lineage with UnityCatalog

