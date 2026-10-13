---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd support 'time travel' queries on the lake so during a postmortem you can see exactly what the data looked like at incident time.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Hot vs Cold Telemetry](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Observability_Data_Lake/Hot_vs_Cold_Telemetry.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To support "time travel" queries in an Observability Data Lake, we need to ensure that we can retrieve historical states of data accurately. This is crucial during postmortems when we need to understand exactly what the state of the system was at a specific point in time. The key idea here involves storing data with timestamps and implementing a robust query mechanism that allows for precise time-based retrieval.

The question is probing our understanding of how to manage and access historical data efficiently while ensuring data integrity and performance. Key assumptions include:

The Data Lake contains time-series data.
There's a need to support both real-time ingestion and historical querying.
We are dealing with potentially large volumes of data, requiring scalable solutions.

We will approach this from first principles by considering the storage strategy, query mechanism, and operational aspects.

1. Storage Strategy

For supporting time travel queries, we should store our data with a timestamp or version number to uniquely identify each state at different points in time. One common approach is using a time-series database like InfluxDB, which natively supports time-based queries. However, for an Observability Data Lake that might handle diverse types of data (not just time series), we can use a hybrid storage strategy.

1.1 Time-Series Data Storage

For time-series metrics, we would store the data in a dedicated time-series database such as InfluxDB or OpenTSDB. Each entry would include a timestamp and any associated metadata.

PLAINTEXT
Measurement: "cpu_usage"
Timestamp: 2023-09-15T14:30:00Z
Fields:
  - value: 78
Tags:
  - host: server1
1.2 Non-Time-Series Data Storage

For non-time-series data, we can use a more general-purpose storage solution like Amazon S3 or Azure Blob Storage. We would store the data with metadata that includes timestamps.

JSON
{
  "timestamp": "2023-09-15T14:30:00Z",
  "data": {
    // Data content here
  }
}
2. Query Mechanism

To support time travel queries, we need a robust query mechanism that can retrieve data based on specific timestamps.

2.1 Time-Series Queries

For time-series data, the InfluxDB query language provides powerful tools to fetch data within a specified time range.

INFLUXQL
SELECT value FROM cpu_usage WHERE host='server1' AND time >= '2023-09-15T14:30:00Z' AND time < '2023-09-15T15:00:00Z'
2.2 Non-Time-Series Queries

For non-time-series data, we can use SQL-like queries on top of the storage solution.

SQL
SELECT * FROM data WHERE timestamp >= '2023-09-15T14:30:00Z' AND timestamp < '2023-09-15T15:00:00Z'
3. Operational Considerations

To ensure that the time travel feature works reliably and efficiently:

Data Retention Policies: Implement policies to retain data for a specific period, ensuring compliance with regulatory requirements.
Indexing: Create indexes on timestamp fields in storage solutions to speed up queries.
Backup and Recovery: Regularly back up historical data to prevent loss of critical information.
4. Scalability

To handle large volumes of data:

Use sharding strategies for time-series data across multiple instances of the database.
Implement a distributed file system for non-time-series data if using object storage solutions.

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, supporting "time travel" queries involves using a combination of specialized time-series databases and general-purpose storage solutions. We need to implement robust query mechanisms that can handle both types of data efficiently. Operational considerations such as data retention policies, indexing, and backup are crucial for ensuring the reliability and performance of our solution.

#### Common mistakes
might include:

Failing to index timestamp fields adequately.
Not implementing proper data retention strategies, leading to storage issues.
Overlooking the need for distributed systems when handling large volumes of data.

In follow-ups, we could explore more advanced topics like using machine learning models to predict future states based on historical data or integrating with logging and tracing systems for a comprehensive observability solution.

**Code:**

```
Example InfluxDB query

SELECT value FROM cpu_usage WHERE host='server1' AND time >= '2023-09-15T14:30:00Z' AND time < '2023-09-15T15:00:00Z'

CODE
Copy

```sql
# Example SQL query for non-time-series data
SELECT * FROM data WHERE timestamp >= '2023-09-15T14:30:00Z' AND timestamp < '2023-09-15T15:00:00Z'
```

**Data:**

```json
{
  "timestamp": "2023-09-15T14:30:00Z",
  "data": {
    // Data content here
  }
}
```

**Flow:**

1. Yes
2. No
3. Time Travel Query Request
4. Is Time-Series Data?
5. Use InfluxDB Query
6. Use SQL-like Storage Query
7. Retrieve Time-Series Data
8. Retrieve Non-Time-Series Data
9. Return Non-Time-Series Data
10. Time Travel Query Response

