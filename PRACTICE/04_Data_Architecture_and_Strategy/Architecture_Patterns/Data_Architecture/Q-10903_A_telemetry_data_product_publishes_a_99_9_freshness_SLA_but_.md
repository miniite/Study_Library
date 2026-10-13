---
type: "practice"
difficulty: "Expert"
---

# A telemetry data product publishes a 99.9% freshness SLA but a critical auto-remediation flow needs 99.99%. Architect SLA composition and dependency risk across a chain of telemetry products in a mesh so the automation team isn't held hostage by a weak upstream.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Data Product SLAs](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Data_Product_SLAs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To architect an SLA-compliant data pipeline while addressing the critical auto-remediation flow's stringent 99.99% freshness requirement, we need to carefully design a mesh of telemetry products and their interdependencies, ensuring that each component contributes to the overall reliability and performance. The key challenge here is balancing the SLAs across different components in a way that minimizes risk while maximizing availability.

#### Triaging the Question

The core question revolves around ensuring that the auto-remediation flow meets its 99.99% freshness requirement, given that other telemetry products only meet a 99.9% SLA. This involves understanding and mapping out the dependencies between these products, identifying potential bottlenecks, and architecting a resilient data mesh.

#### Assumptions
SLAs: Telemetry product A has an 99.9% SLA, while Product B needs to meet a 99.99% SLA.
Data Latency: The auto-remediation flow is sensitive to data latency, and any delays could lead to critical issues.
Failure Modes: We need to account for scenarios where the upstream product might fail or be delayed.

#### First-Principles Solution

Decentralized Data Mesh:

Each telemetry product should have its own data lake or warehouse (e.g., AWS S3, Google BigQuery) and a dedicated stream processing layer.

Data Pipeline Design:

Implement a real-time pipeline to process the 99.9% SLA data as quickly as possible.
Use a batch processing system for the 99.99% SLA requirement to handle any delays gracefully.

Service Level Objectives (SLOs):

Define and monitor SLOs at each stage of the pipeline.
Set up alerts and automated failover mechanisms to address any breaches in SLAs.

Resilient Architecture:

Use redundant components and failover strategies.
Implement retries, backoff policies, and data caching to reduce latency.

Quality of Service (QoS):

Prioritize the 99.99% SLA traffic over other streams using QoS mechanisms in the network or processing pipelines.
Step-by-Step Solution
Data Collection:
Telemetry Product A collects data and stores it in a decentralized lake.
Real-Time Processing:
Use a stream processing framework (e.g., Apache Kafka, AWS Kinesis) to process the 99.9% SLA data in real-time.
Batch Processing:
For the auto-remediation flow, implement batch processing with lower latency but higher accuracy requirements.
Data Aggregation and Storage:
Aggregate and store processed data in a central warehouse (e.g., AWS Redshift).
Monitoring and Alerting:
Set up monitoring for both real-time and batch processes.
Define alert thresholds to notify the operations team of any SLA breaches.

#### Closing Summary

By architecting a resilient mesh of telemetry products with distinct data pipelines and SLOs, we can ensure that the critical auto-remediation flow meets its stringent 99.99% freshness requirement without compromising other products' SLAs. This approach leverages redundancy, QoS mechanisms, and real-time processing to minimize risk.

#### Common Mistakes/Follow-Ups
Over-Reliance on Single Point of Failure: Ensure that the architecture does not rely too heavily on any single component.
Ignoring Latency in Batch Processes: Address potential delays in batch processes by implementing retries and backoff policies.
Underestimating SLA Breaches: Plan for scenarios where SLAs are breached and have failover strategies in place.

**Comparison:**

```
Component	SLA Requirement	Processing Method
Telemetry Product A	99.9% freshness	Real-time streaming
Auto-Remediation Flow	99.99% freshness	Batch processing
```

**Code:**

```python
# Example of a simple stream processing setup using Apache Kafka and Python
from kafka import KafkaProducer

producer = KafkaProducer(bootstrap_servers='localhost:9092')
data = b'{"key": "value"}'
future = producer.send('topic', value=data)
```

**Example input / output:**

```text
Input:
- Telemetry Product A emits data with 99.9% freshness SLA

Output:
1. Real-time processing of Product A's data (99.9% SLA) using Kafka and Python script.
2. Batch processing for auto-remediation flow with SLOs set to 99.99%.
3. Central warehouse stores processed data from both streams.
```

**Data:**

```json
{
  "telemetry_product_a": {
    "data_lake": "s3://bucket/telemetry/product-a",
    "stream_processing_topic": "kafka-topic-a"
  },
  "auto_remediation_flow": {
    "batch_processing_topic": "kafka-topic-b",
    "central_warehouse": "redshift:database-name"
  }
}
```

**Flow:**

1. Telemetry Product A
2. Kafka Topic A
3. Real-Time Processing Layer
4. Central Warehouse (Redshift)
5. Batch Processing Layer forAuto-Remediation
6. Auto-Remediation Flow

