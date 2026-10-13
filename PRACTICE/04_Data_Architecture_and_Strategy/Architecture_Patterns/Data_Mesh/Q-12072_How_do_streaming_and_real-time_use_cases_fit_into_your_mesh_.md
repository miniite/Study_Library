---
type: "practice"
difficulty: "Expert"
---

# How do streaming and real-time use cases fit into your mesh, and what changes about the data product contract when the output port is an event stream?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Data Product Discoverability](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Mesh/Data_Product_Discoverability.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address how streaming and real-time use cases fit into a Data Mesh architecture and the implications on the data product contract, let's start with a brief overview and then delve into the details.

TL;DR

In a Data Mesh, streaming and real-time use cases are integral parts of the data fabric. The key change in the data product contract for event streams is that the output port becomes an unbounded sequence of events rather than a static dataset. This shift requires careful consideration of state management, consistency models, and operational practices to ensure reliability and performance.

#### Triage

The question probes your understanding of Data Mesh principles, specifically how they apply to real-time data processing. The core idea is that the data product contract must be reimagined when the output is an event stream rather than a batch dataset. We will assume you have a basic knowledge of Data Mesh architecture and streaming technologies.

#### First-Principles Solution
Understanding Data Mesh

Data Mesh is a paradigm where data is treated as a product, with each business domain owning its data but also consuming data from other domains. The key tenets include:

Domain Ownership: Each domain owns the data it generates.
Self-service Data Infrastructure: Domains are responsible for their own data infrastructure and governance.
Data Product Contract: A formal agreement between producers and consumers outlining how the data is used.
Streaming vs. Batch Processing

In traditional batch processing, the output is a static dataset that can be processed multiple times. However, in streaming and real-time scenarios, the output is an unbounded sequence of events, which requires different handling:

State Management: In batch processing, state management is straightforward as the data set is finite. In streaming, you need to manage state across events.

#### Cons
istency Models: Batch systems often use eventual consistency models, while real-time systems might require strong or sessional consistency.
Scalability and Performance: Streaming architectures must handle high throughput and low latency.
Data Product Contract for Event Streams

The data product contract changes as follows:

Contract Definition:

Schema Evolution: The schema of the event stream can evolve over time, unlike a static dataset.
Latency Requirements: There is an emphasis on low-latency processing and delivery times.
Reliability: Ensuring that no events are lost or duplicated is crucial.

#### Cons
umer Expectations:

Eventual Consistency: Consumers should expect eventual consistency, especially if the source data has high volatility.
Real-time Updates: Consumers need real-time updates to make timely decisions based on incoming data.

Operational Practices:

Backpressure Handling: Implementing mechanisms to handle backpressure and avoid overloading downstream systems.
Error Handling and Retries: Robust error handling and retry mechanisms are essential for maintaining availability.
Monitoring and Alerting: Continuous monitoring of the streaming pipeline is necessary to detect issues early.

#### Closing Summary

In summary, when integrating real-time use cases into a Data Mesh architecture, the data product contract must account for the dynamic nature of event streams. This includes managing state, ensuring consistency, and adapting operational practices to handle high throughput and low latency requirements.

#### Common Mistakes
and Follow-Ups
Mistake 1: Ignoring state management in favor of simplicity.
Mistake 2: Assuming eventual consistency is sufficient for real-time applications where timely decisions are critical.

#### Follow-Up
Discussing specific tools and technologies used to implement these changes, such as Apache Kafka or AWS Kinesis.

**Comparison:**

| Timestamp         | Value |
|-------------------|-------|
| 2023-10-01 12:00  | 5     |
| 2023-10-01 12:01  | 7     |
| 2023-10-01 12:02  | 9     |
| Timestamp         | Total Value |
|-------------------|-------------|
| 2023-10-01 12:00  | 5           |
| 2023-10-01 12:01  | 12          |
| 2023-10-01 12:02  | 21          |

**Code:**

```python
# Example of a simple stateful processing in Python using Apache Flink
from pyflink.dataset import ExecutionEnvironment
from pyflink.table import StreamTableEnvironment, DataTypes
from pyflink.table.descriptors import Schema, OldCsv, FileSystem

env = ExecutionEnvironment.get_execution_environment()
t_env = StreamTableEnvironment.create(env)

# Define the schema for the input stream
t_env.connect(FileSystem().path('input_path'))
     .with_format(OldCsv()
                  .field('key', DataTypes.STRING())
                  .field('value', DataTypes.INT()))
     .with_schema(Schema()
                  .field('key', 'STRING')
                  .field('value', 'INT'))
     .in_append_mode()
     .register_table_source('mySource')

t_env.connect(FileSystem().path('output_path'))
     .with_format(OldCsv()
                  .field('key', DataTypes.STRING())
                  .field('value', DataTypes.INT()))
     .with_schema(Schema()
                  .field('key', 'STRING')
                  .field('value', 'INT'))
     .in_append_mode()
     .register_table_sink('mySink')

# Define the processing logic
t_env.from_path('mySource') \
    .group_by(t_env.group_id()) \
    .select('key, SUM(value) as total_value') \
    .to_path('mySink')
```

**Example input / output:**

```text
Input Event Stream:
[{"timestamp": "2023-10-01T12:00:00Z", "value": 5},
 {"timestamp": "2023-10-01T12:01:00Z", "value": 7},
 {"timestamp": "2023-10-01T12:02:00Z", "value": 9}]

Output Event Stream:
[{"timestamp": "2023-10-01T12:00:00Z", "total_value": 5},
 {"timestamp": "2023-10-01T12:01:00Z", "total_value": 12},
 {"timestamp": "2023-10-01T12:02:00Z", "total_value": 21}]
```

**Data:**

```json
{
  "data": [
    {"timestamp": "2023-10-01T12:00:00Z", "value": 5},
    {"timestamp": "2023-10-01T12:01:00Z", "value": 7},
    {"timestamp": "2023-10-01T12:02:00Z", "value": 9}
  ]
}
```

**Flow:**

1. Event Source
2. Filter
3. Aggregation
4. Sink
5. Real-time Consumer

