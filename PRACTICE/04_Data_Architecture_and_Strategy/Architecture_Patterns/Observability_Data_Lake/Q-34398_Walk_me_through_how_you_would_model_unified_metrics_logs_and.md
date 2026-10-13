---
type: "practice"
difficulty: "Deep"
---

# Walk me through how you would model unified metrics logs and traces in a data lake so that an AIOps correlation engine can join them at query time.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ In-lake vs External Inference](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Observability_Data_Lake/In-lake_vs_External_Inference.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To model unified metrics, logs, and traces in a data lake for effective AIOps correlation, we need to design a robust schema that captures all necessary data dimensions while ensuring efficient querying capabilities. This involves understanding the nature of each type of data, their relationships, and how they can be joined at query time.

The core idea is to unify these disparate data sources into a common format that allows for seamless correlation without requiring complex transformations during queries. By doing so, we ensure that our AIOps engine can effectively analyze and correlate metrics, logs, and traces on-the-fly, providing real-time insights and actionable intelligence.

Firstly, let's break down the types of data:

Metrics provide quantitative information about system behavior.
Logs capture events and messages from various sources.
Traces offer context for specific requests or operations, often in a distributed environment.

Each type has its unique structure but shares common attributes such as timestamp, entity ID, severity, etc. To unify these, we will design a schema that captures the essence of all three while also allowing flexibility and extensibility.

#### Triage

The interviewer is probing how you would architect a data model for these diverse data types in a way that supports efficient correlation. Key assumptions include:

The AIOps engine needs to correlate metrics, logs, and traces based on common attributes.
The system must handle large volumes of data efficiently.
Flexibility should be maintained to support future additions or changes.
Step-by-Step Solution

Define Common Attributes:

Timestamp: All data points have a timestamp.
Entity ID: Unique identifier for the entity (e.g., host, service).
Severity: For logs and traces indicating importance level.

Schema Design:

SQL
CREATE TABLE UnifiedData (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    entity_id VARCHAR(100),
    timestamp TIMESTAMP NOT NULL,
    severity INT DEFAULT 0,
    metric_value FLOAT,
    log_message TEXT,
    trace_id VARCHAR(100),
    trace_span_id VARCHAR(50),
    -- Other relevant fields
);

Data Ingestion:

Metrics: Direct ingestion from monitoring systems.
Logs: Ingested using structured logging frameworks and transformed into the unified format.
Traces: Collected from distributed tracing systems and normalized.

Joining Data at Query Time:

Use SQL JOINs or semi-structured data querying tools like Apache Drill to correlate metrics, logs, and traces based on entity_id and timestamp.

#### Example
query using SQL-like syntax:
SQL
SELECT m.*, l.*, t.*
FROM UnifiedData m
JOIN UnifiedData l ON m.entity_id = l.entity_id AND m.timestamp = l.timestamp
LEFT JOIN UnifiedData t ON m.trace_id = t.trace_id;

Indexing and Optimization:

Create indexes on entity_id and timestamp to speed up join operations.
Use vector clocks or other mechanisms for distributed tracing correlation.

#### Closing Summary

By modeling unified metrics, logs, and traces in a common format within the data lake, we ensure that our AIOps engine can efficiently correlate these diverse data sources. This approach balances flexibility with performance optimization, allowing us to leverage advanced analytics and machine learning techniques for real-time insights.

#### Common Mistakes/Follow-Ups

Not properly handling time synchronization across different data sources.
Overcomplicating the schema design leading to poor query performance.
Ignoring the importance of entity and timestamp alignment when joining traces with metrics/logs.

**Comparison:**

```
entity_id	timestamp	severity	metric_value	log_message	trace_id
host-001	2023-10-01T12:00:00Z	2	95.6	High CPU usage detected on host-001.	NULL
host-001	2023-10-01T12:00:00Z	2	NULL	NULL	NULL
trc-abc123	2023-10-01T12:00:05Z	2	NULL	NULL	trc-abc123
```

**Code:**

```sql
CREATE TABLE UnifiedData (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    entity_id VARCHAR(100),
    timestamp TIMESTAMP NOT NULL,
    severity INT DEFAULT 0,
    metric_value FLOAT,
    log_message TEXT,
    trace_id VARCHAR(100),
    trace_span_id VARCHAR(50)
);
```

**Example input / output:**

```text
-- UnifiedData m (metrics)
INSERT INTO UnifiedData VALUES ('host-001', '2023-10-01T12:00:00Z', 2, 95.6, NULL, NULL);

-- UnifiedData l (log)
INSERT INTO UnifiedData VALUES ('host-001', '2023-10-01T12:00:00Z', 2, NULL, 'High CPU usage detected on host-001.', NULL);

-- UnifiedData t (trace)
INSERT INTO UnifiedData VALUES ('trc-abc123', '2023-10-01T12:00:05Z', 2, NULL, NULL, 'trc-abc123', 'span-001');

-- Query
SELECT m.*, l.*, t.*
FROM UnifiedData m
JOIN UnifiedData l ON m.entity_id = l.entity_id AND m.timestamp = l.timestamp
LEFT JOIN UnifiedData t ON m.trace_id = t.trace_id;
```

**Data:**

```json
{
    "entity_id": "host-001",
    "timestamp": "2023-10-01T12:00:00Z",
    "severity": 2,
    "metric_value": 95.6,
    "log_message": "High CPU usage detected on host-001.",
    "trace_id": "trc-abc123"
}
```

**Flow:**

1. UnifiedData Metrics
2. (Unified)
3. UnifiedData Logs
4. UnifiedData Traces
5. Correlation Engine

