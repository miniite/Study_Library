---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd use Spark to build a features table from raw metrics for an anomaly model, keeping training and serving features consistent.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Computing SLIs with Spark](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Computing_SLIs_with_Spark.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this question, we need to understand the process of transforming raw metrics into feature vectors that can be used both during model training and in real-time serving. The goal is to ensure consistency between these two stages by leveraging Spark's powerful data processing capabilities, particularly with Kafka for streaming ingestion.

#### Triage

The core challenge here involves integrating a continuous stream of raw metrics into a feature table that can support both training an anomaly detection model and serving predictions in production environments. The key aspects are handling real-time data, ensuring data freshness, managing schema consistency, and optimizing performance.

#### Solution
from First Principles
Data Ingestion: Use Apache Kafka to ingest raw metrics in near-real time.
Structured Processing with Spark Streaming: Apply transformations using Apache Spark Streaming to convert raw metrics into feature vectors.
Feature Table Storage: Store the derived features in a structured format for both training and serving, ensuring consistency.
Serving Layer: Deploy a serving layer that can efficiently retrieve these features during inference.
Step-by-Step Solution

Data Ingestion with Kafka

Set up a Kafka topic to receive raw metrics from sensors or systems.
PYTHON
from kafka import KafkaConsumer

#### cons
umer = KafkaConsumer('metrics_topic', bootstrap_servers=['localhost:9092'])

Structured Processing with Spark Streaming

Initialize a Spark Streaming context and define the processing logic.
SCALA
val sparkConf = new SparkConf().setAppName("FeatureTableBuilder").setMaster("local[*]")
val ssc = new StreamingContext(sparkConf, Seconds(10))

// Define transformations to convert raw metrics into features
val featureDStream = KafkaUtils.createDirectStream[String, String, StringDecoder, StringDecoder] (ssc, PreferredList.empty, consumer)
  .map(_._2)  // Extract message value
  .flatMap(extractFeaturesFromMetric)  // Convert metric string to feature vector

Feature Table Storage

Store the derived features in a structured format like Parquet or Delta Lake for consistency.
SCALA
val featuresDF = featureDStream.mapPartitions(record => {
  record.foreach(f => Seq(f))
}).toDF()

// Save to a table, e.g., using Delta Lake
featuresDF.writeStream.format("delta").option("checkpointLocation", "/path/to/checkpoint").start("/path/to/features_table")
  .awaitTermination()

Serving Layer

Deploy a serving layer (e.g., using Apache Spark MLlib or a custom service) to retrieve and serve features in real-time.
PYTHON
from pyspark.ml.feature import VectorAssembler

# Initialize the model
assembler = VectorAssembler(inputCols=["feature_1", "feature_2"], outputCol="features")
model = SomeTrainedModel.load("/path/to/model")

# Define a serving function
def serve_features(metric_str):
    feature_vector = extractFeaturesFromMetric(metric_str)
    vectorized = assembler.transform([feature_vector])
    prediction = model.predict(vectorized.features)
    return prediction

# Example usage
print(serve_features("some_metric"))

#### Closing Summary
and Common Mistakes/Follow-Ups

This approach ensures that the features used in training are consistent with those served, maintaining model reliability. Key considerations include real-time data handling, schema consistency, and performance optimization. Real-world challenges like network latency, data skew, and fault tolerance must also be addressed.

#### Common mistakes
to avoid:

Inconsistent Data Schemas: Ensure all data transformations maintain the same feature structure.
Lag in Serving Features: Use efficient storage and retrieval mechanisms to minimize latency.
Resource Overhead: Optimize Spark configurations to balance performance and resource usage.

For a deeper dive, we could discuss advanced topics like stateful processing with Spark Streaming, distributed machine learning algorithms for scalability, and integration with other AIOps tools.

**Comparison:**

| Feature | Description |
|---------|-------------|
| feature_1 | First dimension of the metric vector |
| feature_2 | Second dimension of the metric vector |

**Code:**

```

from kafka import KafkaConsumer

consumer = KafkaConsumer('metrics_topic', bootstrap_servers=['localhost:9092'])

CODE
Copy

```scala
val sparkConf = new SparkConf().setAppName("FeatureTableBuilder").setMaster("local[*]")
val ssc = new StreamingContext(sparkConf, Seconds(10))

// Define transformations to convert raw metrics into features
val featureDStream = KafkaUtils.createDirectStream[String, String, StringDecoder, StringDecoder] (ssc, PreferredList.empty, consumer)
  .map(_._2)  // Extract message value
  .flatMap(extractFeaturesFromMetric)  // Convert metric string to feature vector

// Save to a table, e.g., using Delta Lake
val featuresDF = featureDStream.mapPartitions(record => {
  record.foreach(f => Seq(f))
}).toDF()

featuresDF.writeStream.format("delta").option("checkpointLocation", "/path/to/checkpoint").start("/path/to/features_table")
  .awaitTermination()
PYTHON
Copy
from pyspark.ml.feature import VectorAssembler

# Initialize the model
assembler = VectorAssembler(inputCols=["feature_1", "feature_2"], outputCol="features")
model = SomeTrainedModel.load("/path/to/model")

# Define a serving function
def serve_features(metric_str):
    feature_vector = extractFeaturesFromMetric(metric_str)
    vectorized = assembler.transform([feature_vector])
    prediction = model.predict(vectorized.features)
    return prediction

# Example usage
print(serve_features("some_metric"))
```

**Example input / output:**

```text
Input: Raw metric data stream from Kafka topic `metrics_topic`.

Output:
- Feature vectors saved to Delta Lake table `/path/to/features_table`.
- Real-time predictions served by the model.
```

**Data:**

```json
{
  "metrics_topic": {
    "partitions": [
      {"id": 0, "leader": "broker1"},
      {"id": 1, "leader": "broker2"}
    ]
  }
}
```

**Flow:**

1. Streaming Processing
2. Raw Metrics
3. Kafka Consumer
4. Transformed Features
5. Feature Table (Delta Lake)
6. Serving Layer (Spark MLlibModel)
7. Feature Extraction Logic
8. Spark Streaming Context
9. Feature Vector Stream

