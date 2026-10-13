---
type: "practice"
difficulty: "Basic"
---

# What does the bronze layer typically store in medallion architecture?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Lake Catalogs](<../../../../02_Data_Engineering/Storage_and_Lakehouse/Lakehouse/Lake_Catalogs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a Medallion Architecture, data is stored across three layers: Bronze, Silver, and Gold. The bronze layer stores raw, unprocessed data directly ingested from sources. Typically, this includes full load (FL) or incremental load (IL) of data with minimal transformation. The key idea here is that the bronze layer serves as an initial storage point where all dirty, raw, and potentially incomplete data lands before undergoing processing.

The Medallion architecture is designed to provide a structured approach to handling and processing large volumes of data from various sources efficiently. Let's break down the question step-by-step:

Understanding the Bronze Layer: The bronze layer is often used for storing raw data as it comes in, without any transformations or clean-up. It acts like an initial buffer where uncleaned data can be stored before being processed further.

Data Sources and Volume: Raw data from sources such as databases, web logs, IoT devices, or ETL processes typically land directly here. The volume of this raw data can be massive, making it crucial to store it in a place that is scalable and cost-effective.

Key Characteristics:

No Transformation: Data stored here hasn't undergone any transformation or cleaning.
Full Load vs Incremental Load: This layer often contains either full loads (FL) which means the entire dataset is reloaded periodically, or incremental loads (IL) where only new or updated records are processed.

Storage Considerations:

Data Lakes & Lakehouse: In a Data Lake architecture, this layer might involve storing data in its native format using tools like Apache Parquet for efficient storage and retrieval.
Cost-Effectiveness: Given the volume of raw data, storage solutions must be cost-effective. Hadoop Distributed File System (HDFS) or cloud-based object storage services are common choices.

#### Example Scenario

Suppose a retail company is ingesting sales transaction data from multiple stores. Raw sales records with timestamp, product ID, and customer details would land in the bronze layer.

In summary, the bronze layer typically stores raw, unprocessed data directly ingested from sources without any transformations, serving as an initial storage point for all incoming data before it moves to processing layers.

