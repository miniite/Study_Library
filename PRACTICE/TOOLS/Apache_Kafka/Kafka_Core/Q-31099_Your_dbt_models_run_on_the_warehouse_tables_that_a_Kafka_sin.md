---
type: "practice"
difficulty: "Intermediate"
---

# Your dbt models run on the warehouse tables that a Kafka sink lands into; how do you make dbt runs consistent given the underlying table is being continuously appended?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Exactly Once Beyond Kafka](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Exactly_Once_Beyond_Kafka/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure consistency in dbt model runs when working with a Kafka-sourced warehouse, the key challenge lies in synchronizing the dbt execution timing with the data landing process in the warehouse tables. This involves handling the continuous nature of Kafka streams and ensuring that all relevant data is included in each dbt run.

#### Triage
What's Being Probed

The question revolves around ensuring consistency between dbt model runs and the continuously updating data from a Kafka sink. The interviewer is probing your understanding of:

Data ingestion patterns and their impact on ETL processes.
Techniques for managing incremental or full-refresh dbt models.
Strategies to avoid double-counting or missing data during runs.

#### First Principles Solution

To address this challenge, we can employ several strategies:

Watermarking: This involves tracking the time-stamped data in Kafka to ensure that only new and relevant records are processed by dbt.
Incremental vs Full Refresh Models: Choosing the right type of dbt model (incremental or full-refresh) based on your use case.
Batch Processing Synchronization: Ensuring that the dbt run is synchronized with the Kafka batch processing cycle to avoid overlap.
Implementing Watermarking

Watermarking involves setting a timestamp column in your source data and tracking this timestamp to filter out old records during each dbt run. This ensures that only new data is processed, maintaining consistency.

Step-by-Step Solution
Set Up Timestamp Columns: Ensure all relevant tables in the warehouse have a timestamp or created_at column.
Define Watermarks in Kafka: Configure your Kafka sink to include a watermark timestamp with each message.
Use dbt's incremental Model Type: Use an incremental model in dbt to ensure that only new data is processed.
SQL
-- Example of a dbt incremental model
{{ config(
    materialized='incremental',
    unique_key='unique_id'
) }}

with latest_data as (
  select *
  from {{ source('kafka_source', 'raw_table') }}
  where timestamp > {{ var("last_run_timestamp") }}
)

select *
from latest_data;
Maintain a State Variable: Keep track of the last processed timestamp in a state variable or database table, which dbt can reference to filter data.
Synchronize with Kafka Batch Processing: Ensure that the dbt run timing aligns with the Kafka batch processing schedule.

#### Closing Summary
and Common Mistakes/Follow-Ups

To summarize, using watermarking is crucial for maintaining consistency when working with Kafka-sourced data in dbt. Key takeaways include:

Watermark Configuration: Properly configuring watermarks to track timestamped data.
Incremental Models: Utilizing incremental models in dbt to process only new records.
State Management: Maintaining a state variable to synchronize the last processed timestamp.

#### Common mistakes
include:

Overlooking the need for watermarking, leading to processing old or duplicate data.
Failing to properly configure incremental model configurations in dbt.

For follow-ups, the interviewer might ask about:

#### Performance considerations
of incremental vs. full-refresh models.
Handling scenarios with frequent but small updates versus infrequent but large updates.
Integration with other tools like Apache Flink for stream processing.

**Comparison:**

| unique_id | timestamp   |
|-----------|-------------|
| 1         | 2023-10-01T09:30:00Z |
| 2         | 2023-10-01T10:00:00Z |

**Code:**

```sql
-- Example dbt model using incremental materialization
{{ config(
    materialized='incremental',
    unique_key='unique_id'
) }}

with latest_data as (
  select *
  from {{ source('kafka_source', 'raw_table') }}
  where timestamp > {{ var("last_run_timestamp") }}
)

select *
from latest_data;
```

**Example input / output:**

```text
# Example input and output for dbt run synchronization

Input:
- Last processed timestamp = 2023-10-01T09:00:00Z

Output (dbt model run):
- Processed records with timestamps > 2023-10-01T09:00:00Z
```

**Data:**

```yaml
# Example Kafka topic configuration with watermark
topic: raw_table
timestamp_field: timestamp
primary_key: unique_id
```

**Flow:**

1. Start
2. Kafka Source
3. Warehouse Table
4. dbt Model Run
5. Filter New Data withWatermark
6. Insert into Target
7. End

