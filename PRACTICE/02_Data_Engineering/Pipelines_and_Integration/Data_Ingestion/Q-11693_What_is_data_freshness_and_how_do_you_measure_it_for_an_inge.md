---
type: "practice"
difficulty: "Intermediate"
---

# What is data freshness and how do you measure it for an ingestion pipeline?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ CDC vs Full Reload](<../../../../02_Data_Engineering/Pipelines_and_Integration/Data_Ingestion/CDC_vs_Full_Reload.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Data freshness refers to how recent or up-to-date the data in a system is, ensuring that decisions based on this data are made with as much current information as possible. In the context of data ingestion pipelines, it's crucial to measure and maintain the freshness of data from various sources like databases, APIs, or file systems. This ensures that your analytics and reporting are using the most recent data.

#### Triaging the Question

The question is essentially asking you to explain what data freshness means in a data engineering context and how to monitor it for an ingestion pipeline. The interviewer might be probing your understanding of different metrics used to measure freshness, such as latency or staleness, and your ability to implement these metrics using tools like Airbyte or Fivetran.

#### Assumptions
Pipeline Context: We are considering a data pipeline that ingests raw data from multiple sources into a centralized storage.
Tools Familiarity: You should be familiar with tools like Airbyte or Fivetran for ingestion, and possibly other monitoring tools.
Metrics Understanding: Knowledge of common freshness metrics such as latency (time between the source event and arrival in the target) and staleness (time since the last update).

#### Solution
1. Defining Data Freshness

Data freshness can be measured by several key metrics:

Latency: The time difference between when a new record is created at the source and its appearance in the destination.
Staleness: The duration since the last update to a piece of data.
2. Measuring Latency and Staleness

To measure these, we can use various techniques:

Polling and Event-Driven Approaches:
Polling: Periodically check for new data at sources.
Event-driven: Trigger actions based on events (e.g., file creation, database transactions).
3. Using Airbyte or Fivetran

Both tools offer built-in capabilities to measure freshness but require setup and monitoring.

Airbyte:

Latency: Configure intervals for polling and set up scheduled jobs.
Staleness: Use version control in the source to track changes and compare with destination.

Fivetran:

Latency: Monitor sync frequency settings.
Staleness: Implement data lineage tracking and use Fivetran’s monitoring features.
4. Implementation Example

Let's assume we have a pipeline using Airbyte that syncs data from multiple databases into a central warehouse:

Source Setup:

JSON
{
  "connectionId": "abc123",
  "syncCatalog": [
    {
      "streams": [
        {"name": "users", "replicationKey": "id"},
        {"name": "orders", "replicationKey": "order_id"}
      ]
    }
  ],
  "destinationName": "my-database"
}

Monitoring Freshness:

Latency: Set up periodic checks to log the timestamp of each sync.
PYTHON
import time

start_time = time.time()
# Airbyte sync operation here
end_time = time.time()

latency = end_time - start_time
print(f"Sync Latency: {latency} seconds")

Staleness:

Use a metadata table to track the last update times in both source and destination.
SQL
CREATE TABLE IF NOT EXISTS sync_status (
  stream_name VARCHAR(255),
  last_synced_at TIMESTAMP
);

UPDATE sync_status SET last_synced_at = NOW() WHERE stream_name = 'users';
5. Common Mistakes and Follow-ups
Ignoring Source-Specific Latency: Not considering the time it takes for data to propagate from a source (e.g., network delay, processing time).
Overreliance on Monitoring Tools: Relying solely on monitoring tools without understanding their limitations.
Not Regularly Reviewing Freshness Metrics: Neglecting to set up alerts or regularly review freshness metrics can lead to unnoticed issues.

#### Closing Summary

Measuring data freshness is critical for ensuring the accuracy and relevance of your analytics. Tools like Airbyte and Fivetran provide robust mechanisms, but effective monitoring requires careful setup and regular review.

**Comparison:**

| stream_name | last_synced_at |
|------------|---------------|
| users       |                |
| orders      |                |

**Code:**

```

import time

start_time = time.time()

Simulated Airbyte sync operation

time.sleep(30) # Simulating a 30-second sync process end_time = time.time()

latency = end_time - start_time print(f"Sync Latency: {latency} seconds")

CODE
Copy

```sql
CREATE TABLE IF NOT EXISTS sync_status (
  stream_name VARCHAR(255),
  last_synced_at TIMESTAMP
);

UPDATE sync_status SET last_synced_at = NOW() WHERE stream_name = 'users';
```

**Example input / output:**

```text
Input:
Airbyte configuration file for syncing users and orders from a database.

Output:
{
  "connectionId": "abc123",
  "syncCatalog": [
    {
      "streams": [
        {"name": "users", "replicationKey": "id"},
        {"name": "orders", "replicationKey": "order_id"}
      ]
    }
  ],
  "destinationName": "my-database"
}
```

**Data:**

```json
{
  "connectionId": "abc123",
  "syncCatalog": [
    {
      "streams": [
        {"name": "users", "replicationKey": "id"},
        {"name": "orders", "replicationKey": "order_id"}
      ]
    }
  ],
  "destinationName": "my-database"
}
```

**Flow:**

1. @startuml
2. :start;
3. :Simulate Airbyte sync operation with 30-second delay;
4. :update sync_status table for 'users' stream with current timestamp;
5. :end;
6. @enduml

