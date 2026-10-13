---
type: "practice"
difficulty: "Deep"
---

# How do you design incremental materialized views over the lake for the top 50 alert queries without re-scanning everything each run?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Exactly Once into Observability Lakes](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Observability_Data_Lake/Exactly_Once_into_Observability_Lakes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design incremental materialized views (MV) over an Observability Data Lake for the top 50 alert queries, we need to balance between freshness and cost. The key is to ensure that we efficiently update only those portions of the data lake that have changed since the last run while keeping overhead low. Here’s a step-by-step approach to achieve this:

TL;DR: We will design an incremental MV strategy using change data capture (CDC) techniques, combined with a materialized view pattern that updates only relevant parts of the query results.

Problem and Probing:

The question is asking for a method to maintain up-to-date insights into alert queries without full re-scans.
The interviewer is probing your understanding of data lake architectures, change data capture (CDC), and materialized views.

#### Assumptions
include a data lake with structured logs or events, a database system supporting CDC, and the ability to implement custom triggers.

Step-by-Step Solution:

Data Sources: Assume we have a data lake with streaming or batch ingestion of log data.
CDC Mechanism: Implement CDC on the source systems to capture changes in near real-time. This can be done using change data capture technologies like Apache Kafka’s Debezium connector, AWS DMS, or similar.
Materialized View Design:
Identify Key Metrics and Aggregations: Determine which aggregations or metrics are relevant for the top 50 alert queries.
Partitioning Strategy: Use partitioning to segment data based on time or other dimensions, allowing incremental updates.
Incremental Update Logic:
For each alert query, apply a custom SQL trigger that only touches the necessary partitions based on change timestamps.
Leverage database-specific features like MERGE statements for efficient upsert operations.
Change Tracking: Maintain a changelog table or use database-specific features to track changes. This helps in determining which parts of the data need updating.

#### Closing Summary
and Common Mistakes:

Incremental MVs reduce the computational load significantly by only processing changed data.

#### Common mistakes
include over-complicating the CDC setup, not leveraging built-in database functionalities for efficiency, or neglecting to properly partition data.

**Comparison:**

| Alert Query | Description |
|-------------|-------------|
| 1           | Summarize top alerts by time period |
| 2           | Count unique alert types per hour |
| Change Tracking Schema |
|------------------------|
| id (unique identifier) |
| timestamp              |
| change_type            |
| new_values             |

**Code:**

```sql
-- Example SQL Trigger to handle incremental updates
CREATE OR REPLACE TRIGGER alert_query_update_trigger
AFTER INSERT OR UPDATE OR DELETE ON alerts
FOR EACH ROW
BEGIN
    -- Update relevant partitions in the materialized view
    MERGE INTO materialized_view_name mv
    USING (SELECT * FROM changes WHERE change_timestamp > :old.change_timestamp) src
    ON ...
    WHEN MATCHED THEN UPDATE SET ...
    WHEN NOT MATCHED THEN INSERT ...;
END;
```

**Example input / output:**

```text
Input (Initial State):
- Alerts table with existing data

Output (After Change Detection and Update):
- Materialized view updated only for affected partitions or rows
- Efficient use of database resources
```

**Data:**

```json
{
  "alerts": {
    "id": "12345",
    "timestamp": "2023-10-01T00:00:00Z",
    "change_timestamp": "2023-10-02T12:34:56Z",
    "metric_value": 10.5
  },
  "changes": {
    "id": "12345",
    "timestamp": "2023-10-02T12:34:57Z",
    "change_type": "UPDATE",
    "new_values": {"metric_value": 11.0}
  }
}
```

**Flow:**

1. Data Ingestion
2. Change Data Capture
3. Materialized View UpdateTrigger
4. Incremental Update Logic
5. Merge Statement for EfficientUpsert
6. Updated Materialized View

