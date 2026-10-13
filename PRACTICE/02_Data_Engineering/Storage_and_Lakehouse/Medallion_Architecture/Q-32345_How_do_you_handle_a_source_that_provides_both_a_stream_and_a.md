---
type: "practice"
difficulty: "Deep"
---

# How do you handle a source that provides both a stream and a nightly batch of the same data? Which becomes Bronze, and how do you reconcile the two feeds?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Medallion Layer Quality](<../../../../02_Data_Engineering/Storage_and_Lakehouse/Medallion_Architecture/Medallion_Layer_Quality.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of handling a source that provides both a stream (real-time) and a nightly batch feed of the same data in a Medallion Architecture, we need to consider how each feed contributes to our bronze layer. The primary goal is to ensure data integrity, consistency, and efficient processing. In this scenario, the stream will likely become Bronze (raw, unprocessed), while the nightly batch might be used as a more complete source for creating Silver (transformed) or Gold (aggregated) layers.

#### Triage

The question asks how to handle real-time and batch feeds from the same source in a Medallion Architecture. The interviewer is probing your understanding of data processing workflows, particularly how to reconcile live data streams with historical batches. Key assumptions include that both sources provide overlapping but potentially differing data at different timescales.

#### First-Principles Solution
Bronze Layer: This layer contains the raw, unprocessed data. For our scenario, this will be the real-time stream.
Silver Layer: This layer involves processing the Bronze data to produce a more structured and consistent dataset. We can use both the streaming and batch feeds here to create a comprehensive dataset.
Gold Layer: This layer aggregates insights from the Silver layer, providing high-level summaries and analyses.
Step-by-Step Solution

Stream Processing:

Configure your stream processing pipeline (e.g., Apache Kafka, AWS Kinesis) to ingest real-time data as it arrives.
Store this raw data directly in the Bronze Layer database or storage system (e.g., Amazon S3 for Parquet files).

Batch Processing:

Set up a nightly batch job that processes historical and current data from both the stream and the latest batch feed.
Use the batch processing to update the Silver Layer with more complete, transformed data.
Reconciliation Strategy
Version Control: Maintain version control for both streams and batches. This helps in tracking changes and resolving conflicts if they arise.
Event Time Handling: Implement event time processing strategies (e.g., watermarking) to ensure that streaming data is processed based on its actual timestamp, not ingestion order.
Incremental Processing: For the batch job, use incremental updates to avoid overwriting recent stream data. This can be achieved by marking records as "processed" in the Bronze Layer.

#### Implementation
Details

Bronze Layer:

PLAINTEXT
# Example Schema for Bronze Layer (Parquet)
CREATE TABLE bronze_data (
    id INT,
    timestamp TIMESTAMP,
    value FLOAT,
    processed BOOLEAN DEFAULT FALSE  # Indicates if this record was used by batch processing
);

Silver Layer:

Use a dataflow framework like Apache Flink or Beam to process the Bronze stream and batch feeds.
PYTHON
from apache_beam import Pipeline, ParDo, ReadFromKafka

p = Pipeline()

# Read from Kafka (streaming)
stream_source = p | 'Read Stream' >> ReadFromKafka(consumer_config={'bootstrap.servers': 'localhost:9092'},
                                                   topics=['my_topic'])

batch_feed = p | 'Read Batch Feed' >> ReadFromStorage('s3://my-bucket/')

# Process both sources together
processed_data = (stream_source, batch_feed) | 'Merge Streams and Batches' >> MergeStreamsAndBatches()

_ = processed_data | 'Transform Data' >> Transform()

#### Closing Summary

By handling the stream and nightly batch feeds separately in a Medallion Architecture, we ensure data integrity and efficient processing. The Bronze Layer captures raw data, while the Silver Layer reconciles and processes it more comprehensively.

#### Common Mistakes/Follow-Ups
Mistake: Overwriting recent stream data during batch processing without proper version control.

#### Follow-Up
How would you handle conflicts between stream and batch updates? Consider implementing a conflict resolution strategy or using a database with built-in support for upsert operations.

**Comparison:**

|Bronze Layer Schema|
|---|
|id (INT)|
|timestamp (TIMESTAMP)|
|value (FLOAT)|
|processed (BOOLEAN)|

**Code:**

```python
from apache_beam import Pipeline, ParDo, ReadFromKafka

p = Pipeline()

# Read from Kafka (streaming)
stream_source = p | 'Read Stream' >> ReadFromKafka(consumer_config={'bootstrap.servers': 'localhost:9092'},
                                                  topics=['my_topic'])

batch_feed = p | 'Read Batch Feed' >> ReadFromStorage('s3://my-bucket/')

# Process both sources together
processed_data = (stream_source, batch_feed) | 'Merge Streams and Batches' >> MergeStreamsAndBatches()

_ = processed_data | 'Transform Data' >> Transform()
```

**Example input / output:**

```text
- **Input**: Real-time data stream and nightly batch feed.
- **Output**: Stream data stored in Bronze Layer, transformed data in Silver Layer.
```

**Data:**

```plaintext
CREATE TABLE bronze_data (
    id INT,
    timestamp TIMESTAMP,
    value FLOAT,
    processed BOOLEAN DEFAULT FALSE  # Indicates if this record was used by batch processing
);
```

**Flow:**

1. Real-time Stream
2. Bronze Layer Storage
3. Nightly Batch Feed
4. Silver Layer Processing
5. Silver Layer Schema
6. Incremental Update Logic
7. Silver Layer Aggregation

