---
type: "practice"
difficulty: "Expert"
---

# Design a hybrid architecture where the write path lands in a wide-column store for scale but a relational read-model is maintained via CDC for complex reporting. Justify the added complexity, and describe how you keep the two from drifting.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Wide Column Design and Migration](<../../../../01_Databases_and_SQL/NoSQL_and_NewSQL/Wide_Column_Stores/DynamoDB_Modeling_and_Capacity/Wide_Column_Design_and_Migration.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary

In this hybrid architecture design, we leverage Cassandra/DynamoDB as the primary wide-column store for high-scale write operations while maintaining a relational read-model via CDC (Change Data Capture) to support complex reporting queries. This approach balances performance and data integrity but introduces complexity in managing two separate database systems.

#### Triaging the Question

The core of this question revolves around designing an efficient hybrid architecture that combines the strengths of Cassandra/DynamoDB for high-scale writes with a relational read-model for complex reporting. The key challenges are ensuring data consistency between the two stores and optimizing performance while minimizing maintenance overhead.

#### Assumptions
Data Volume: High write volume is expected, but lower complexity reads.
Reporting Queries: Complex SQL-based queries for detailed analytics.
Latency Tolerance: Acceptable latency for writes, critical for reads due to reporting requirements.
Step-by-Step Solution
1. Write Path Design

Cassandra/DynamoDB (Wide Column Store)

Use Case: High write throughput and low-latency writes.
Data Model: Wide columns for fast updates and efficient storage.

#### Implementation
Direct writes from the application to Cassandra/DynamoDB.
Application
Cassandra/DynamoDB
2. Read Path Design

Relational Database (Transactional Store)

Use Case: Complex reporting queries with ACID properties.
Data Model: Traditional relational schema for structured querying.

#### Implementation
CDC to capture changes from Cassandra/DynamoDB and replicate them into the relational store.
Cassandra/DynamoDB
Relational DB
Reporting Service
3. CDC Mechanism

Change Data Capture (CDC)

#### Implementation
Using tools like AWS DMS, Kafka Streams, or custom logic to capture changes.
Process:
Detect write operations in Cassandra/DynamoDB.
Replicate the changes using a CDC tool.
Apply the changes to the relational database.
Cassandra/DynamoDB
Change Data Capture Tool
Relational DB
4. Ensuring Consistency

#### Cons
istency Model

Bounded Staleness: Accept a small degree of staleness in the read model for high write throughput.
Validation Layer: Implement a validation layer to ensure that critical updates are reflected correctly.
Cassandra/DynamoDB
Validation Layer
Relational DB
5. Performance Optimization

Optimization Strategies

Partitioning and Indexing: Optimize Cassandra/DynamoDB with appropriate partition keys and indexes.
Query Optimization: Use SQL queries in the relational database to leverage indexes for fast reporting.
Cassandra/DynamoDB
Partitioning & Indexing
C
SQL Queries

#### Closing Summary

This hybrid architecture design leverages Cassandra/DynamoDB for high-scale writes and maintains a relational read-model via CDC to support complex reporting. The added complexity involves managing two database systems, ensuring data consistency, and optimizing performance. However, the benefits of this approach are significant: high write throughput, low-latency writes, and robust reporting capabilities.

#### Common Mistakes/Follow-Ups
Data Drift: Ensure frequent validation to avoid data inconsistencies.
Latency Trade-offs: Balance between write latency in Cassandra/DynamoDB and read performance in the relational database.
Maintenance Overhead: Regularly monitor CDC replication for any bottlenecks or issues.

**Comparison:**

```
Operation	Cassandra/DynamoDB	Relational DB
Write	High write throughput, low-latency writes	Low priority for writes
Read	Fast reads using wide columns	Complex SQL queries with ACID properties
```

**Code:**

```python
# Example Python script for CDC implementation using Kafka Streams
from kafka import KafkaProducer, KafkaConsumer

producer = KafkaProducer(bootstrap_servers=['localhost:9092'])
consumer = KafkaConsumer('CDC Topic', bootstrap_servers=['localhost:9092'])

for message in consumer:
    data = json.loads(message.value)
    # Apply changes to the relational database
```

**Example input / output:**

```text
Input:
- Write operation to Cassandra/DynamoDB with CDC enabled

Output:
- Changes captured in 'user_activity_changes' topic on Kafka
- Replication of changes into the relational database for reporting
```

**Data:**

```json
{
  "write_operation": {
    "table_name": "user_activity",
    "columns": ["user_id", "activity_type", "timestamp"]
  },
  "cdc_topic": {
    "name": "user_activity_changes",
    "partition_key": "user_id"
  }
}
```

**Flow:**

1. Application
2. Cassandra/DynamoDB
3. Change Data Capture Tool
4. Kafka Topic(user_activity_changes)
5. Relational DB
6. Reporting Service
7. Database Queries

