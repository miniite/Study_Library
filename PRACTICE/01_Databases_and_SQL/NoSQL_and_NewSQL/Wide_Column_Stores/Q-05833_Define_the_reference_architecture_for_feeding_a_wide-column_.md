---
type: "practice"
difficulty: "Expert"
---

# Define the reference architecture for feeding a wide-column operational store into an analytics lakehouse without hammering production. Compare DynamoDB S3 export + Athena, Streams-to-Kinesis-to-warehouse, and Cassandra CDC, and pick a default with justification.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Wide Column Design and Migration](<../../../../01_Databases_and_SQL/NoSQL_and_NewSQL/Wide_Column_Stores/DynamoDB_Modeling_and_Capacity/Wide_Column_Design_and_Migration.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To define a reference architecture that feeds a wide-column operational store (Cassandra or DynamoDB) into an analytics lakehouse without hammering production, we need to consider both the data transfer mechanisms and the processing strategies involved. The goal is to minimize performance impact on the production system while ensuring efficient data ingestion into the analytics environment. We will compare three approaches: DynamoDB S3 Export + Athena, Streams-to-Kinesis-to-Warehouse (KTSW), and Cassandra CDC.

TL;DR

Each approach has its own strengths and weaknesses:

DynamoDB S3 Export + Athena: Simple to set up but may introduce latency.
Streams-to-Kinesis-to-Warehouse (KTSW): Offers real-time data streaming, but requires more complex setup and maintenance.
Cassandra CDC: Provides near-real-time updates with minimal changes required on the operational store.

We will choose DynamoDB S3 Export + Athena as the default due to its simplicity and robustness in handling large-scale data ingestion without significant performance overhead on production systems.

#### Triage

The question is probing our ability to understand different data export strategies for wide-column databases, and how to balance real-time requirements with operational constraints. We need to make assumptions about the scale of the dataset, the frequency of updates, and the desired latency in analytics queries.

#### Assumptions
The production system uses DynamoDB as the operational store.
The analytics requirement demands periodic analysis rather than real-time insights.
The size of the dataset is substantial (millions or billions of records).

#### First Principles Solution

DynamoDB S3 Export + Athena:

#### Overview
This approach involves periodically exporting data from DynamoDB to an S3 bucket, and then querying this exported data using AWS Athena for analytics.
Setup:
BASH
# Enable S3 Export in DynamoDB
aws dynamodb enable-streaming --table-name <TableName> --stream-attributes NEW_IMAGE

# Configure S3 export destination
aws dynamodb describe-table --table-name <TableName>

Streams-to-Kinesis-to-Warehouse (KTSW):

#### Overview
This involves setting up a DynamoDB stream to capture changes, using Kinesis Data Streams for event buffering, and then ingesting these events into an analytics warehouse like Redshift or Snowflake.
Setup:
BASH
# Enable DynamoDB Streams
aws dynamodb enable-streaming --table-name <TableName> --stream-attributes NEW_IMAGE

# Create Kinesis Data Stream
aws kinesis create-stream --stream-name <StreamName>

# Set up a Kinesis Data Firehose delivery stream to write data to the warehouse
aws firehose create-delivery-stream --delivery-stream-name <FirehoseStreamName> --s3-destination-configuration ContentCompressionType=Zipped,RoleARN=<RoleArn>,BucketARN=<BucketArn>,Prefix=<Prefix>

Cassandra CDC:

#### Overview
This approach involves using Apache Cassandra’s Change Data Capture (CDC) to stream changes directly into an analytics warehouse.
Setup:
BASH
# Configure CDC in Cassandra
nodetool enablecdc

# Set up a Kafka cluster for CDC streaming
# Connect Cassandra CDC to Kafka topics and consume those topics in the analytics environment
Comparison and Justification

Simplicity and Robustness: DynamoDB S3 Export + Athena is simpler to set up and maintain. The latency introduced by exporting data to S3 can be managed, ensuring that it does not significantly impact production performance.

Real-Time Capabilities: KTSW offers real-time data streaming but requires more setup and maintenance. It might introduce additional costs due to the complexity of managing Kinesis Data Streams.

#### Performance Considerations
Cassandra CDC provides near-real-time updates with minimal changes required on the operational store, making it ideal for applications that require frequent data modifications. However, it may be overkill if real-time analytics are not a strict requirement.

#### Closing Summary

Given the requirements and constraints outlined, DynamoDB S3 Export + Athena is the default choice due to its simplicity in setup, robustness, and ability to handle large-scale datasets efficiently without significant performance overhead on the production system. However, for applications that require real-time insights, KTSW or Cassandra CDC should be considered based on specific needs.

#### Common Mistakes/Follow-Ups
Latency: Ensure that the export frequency in DynamoDB S3 Export + Athena is optimized to meet the latency requirements of analytics queries.
Cost: Consider the cost implications of KTSW, especially if real-time processing and buffering are needed.
Data Consistency: Verify the data consistency between the operational store and the analytics environment in both CDC-based approaches.

**Comparison:**

| Approach             | Complexity | Real-Time Capabilities | Latency | Cost |
|----------------------|------------|------------------------|---------|------|
| DynamoDB S3 Export + Athena | Low         | No                     | High    | Low  |
| Streams-to-Kinesis-to-Warehouse (KTSW) | Medium      | Yes                    | Medium  | High |
| Cassandra CDC        | High       | Yes                    | Low     | High |

**Code:**

```bash
# DynamoDB S3 Export + Athena setup example

aws dynamodb enable-streaming --table-name MyTable --stream-attributes NEW_IMAGE

aws s3api put-object-acl --bucket my-bucket --key path/to/exported_data.csv --acl public-read
```

**Example input / output:**

```text
Input (AWS CLI commands):
aws dynamodb enable-streaming --table-name MyTable --stream-attributes NEW_IMAGE

Output:
{
    "StreamEnabled": true,
    "StreamArn": "arn:aws:dynamodb:us-west-2:123456789012:table/MyTable/stream/2023-10-05T09:45:30.123Z/0b37cde4-d3f6-4e87-b9cd-75e6f8a12d45"
}
```

**Data:**

```json
{
  "TableName": "MyTable",
  "StreamArn": "arn:aws:dynamodb:us-west-2:123456789012:table/MyTable/stream/2023-10-05T09:45:30.123Z/0b37cde4-d3f6-4e87-b9cd-75e6f8a12d45",
  "S3Bucket": "my-bucket"
}
```

**Flow:**

1. DynamoDB
2. Enable S3 Export
3. S3 Bucket Setup
4. Athena Table Creation
5. Query Data with Athena
6. Kinesis Data Stream
7. Setup Kinesis Firehose toWarehouse
8. Cassandra CDC
9. Kafka Cluster Setup
10. Consume CDC Data inAnalytics Environment

