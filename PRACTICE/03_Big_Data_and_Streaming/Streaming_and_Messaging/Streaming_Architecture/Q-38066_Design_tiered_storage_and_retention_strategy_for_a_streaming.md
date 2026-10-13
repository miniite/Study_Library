---
type: "practice"
difficulty: "Expert"
---

# Design tiered storage and retention strategy for a streaming backbone that must serve both real-time consumers and multi-year replay for compliance. How do you keep costs sane while making arbitrary historical reprocessing possible?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Graceful Degradation in Streaming](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Provisional_and_Final_Results/Graceful_Degradation_in_Streaming.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a tiered storage and retention strategy for a streaming backbone that serves both real-time consumers and supports multi-year replay for compliance, we must consider multiple factors including cost efficiency, data availability, reliability, and scalability. The key is to balance these requirements while ensuring that the system can handle arbitrary historical reprocessing without incurring excessive costs.

#### Triage

The question asks us to design a storage strategy for a streaming backbone that needs to support both real-time and multi-year replay use cases. This involves:

Ensuring real-time data availability.
Retaining data for compliance purposes over extended periods.
Maintaining cost-effectiveness by efficiently managing storage tiers.

#### First Principles Solution

Data Classification: Categorize data into different classes based on its importance and access patterns.

Tiered Storage Layers:

Hot Storage Layer: For real-time consumers, this layer stores the most recent data with high performance requirements.
Warm Storage Layer: Stores data that needs to be accessed within a specific window but not immediately.
Cold Storage Layer: Stores archival and compliance data, optimized for long-term retention at lower costs.

RetentionPolicy:

Implement policies based on data age or access frequency to move data between storage tiers automatically.

Data Reprocessing Capabilities:

Ensure the system can reprocess historical data efficiently by leveraging batch processing tools and indexing techniques.
Detailed Solution
Data Classification

We classify data into three categories:

Hot Data: Real-time data that needs immediate access for real-time consumers (e.g., last 30 days).
Warm Data: Historical data accessed within a defined window (e.g., the past year but not beyond).
Cold Data: Compliance and archival data stored for long-term retention.
Tiered Storage Layers

Hot Storage Layer:

Use in-memory databases or SSDs to ensure low-latency access.

#### Example
Apache Kafka, Redis, or Memcached.

Warm Storage Layer:

Employ durable storage with moderate latency but better cost-effectiveness.

#### Example
Amazon DynamoDB, Cassandra, or HBase.

Cold Storage Layer:

Utilize object storage services optimized for long-term retention and lower costs.

#### Example
AWS S3 Infrequent Access (IA) or Glacier, Google Cloud Storage Nearline.
RetentionPolicy

Implement a policy-driven approach to move data between tiers automatically:

Hot to Warm: Move data from the hot layer to warm once it's no longer needed for real-time consumption but still requires access within the defined window.
Warm to Cold: Archive warm data into cold storage after it exceeds the retention period for compliance.
Data Reprocessing Capabilities

To support arbitrary historical reprocessing, we need:

Event Sourcing:
Capture all events at the source and store them in a replayable format.
Indexing:
Implement efficient indexing to quickly locate specific data segments during reprocessing.
Batch Processing Frameworks:
Use batch processing tools like Apache Spark or Flink for large-scale historical reprocessing.

#### Example
Architecture
Real-time Consumers
Hot Storage Layer
Warm Storage Layer
Cold Storage Layer
Data Generator
RetentionPolicy
Move to Warm
Move to Cold
Compliance System
Cost Management

To keep costs sane, we can:

Implement automated tiering policies.
Use cost-effective long-term storage solutions for cold data.
Optimize indexing and querying mechanisms to reduce read times.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, the solution involves a tiered approach that categorizes data based on its access patterns and retention needs. By leveraging different storage layers with varying costs, we can ensure both real-time performance and long-term compliance at an acceptable cost. Common mistakes include not properly classifying data or overprovisioning hot storage resources.

For follow-ups, one might need to discuss specific technologies used for each tier, additional security measures, disaster recovery plans, and the impact of network latency on real-time consumers.

**Comparison:**

```
Tier	Storage Type	Retention Period
Hot	SSD	< 1 day
Warm	HDD	1 year - 5 years
Cold	S3 IA	> 5 years
```

**Code:**

```python
# Example Python code for moving data between tiers
def move_data_to_warm(data_id):
    # Logic to move data from hot storage to warm storage
    pass

def move_data_to_cold(data_id):
    # Logic to archive data into cold storage
    pass
```

**Example input / output:**

```text
Input: Move data ID-12345 from hot storage to warm storage.
Output: Data moved successfully to warm storage.

Input: Archive data ID-67890 into cold storage after it exceeds the retention period.
Output: Data archived in S3 IA with metadata for future retrieval.
```

**Data:**

```json
{
  "data_classification": {
    "hot": ["last_30_days"],
    "warm": ["yearly_window"],
    "cold": ["compliance"]
  },
  "storage_tiers": [
    {"layer": "hot", "type": "SSD"},
    {"layer": "warm", "type": "HDD"},
    {"layer": "cold", "type": "S3 IA"}
  ]
}
```

**Flow:**

1. < 1 year
2. > 1 year
3. Real-time Data
4. Hot Storage
5. Data Age
6. Stay in Hot
7. Warm Storage
8. Arbitrary HistoricalReprocessing
9. Batch Processing
10. Indexing & Querying

