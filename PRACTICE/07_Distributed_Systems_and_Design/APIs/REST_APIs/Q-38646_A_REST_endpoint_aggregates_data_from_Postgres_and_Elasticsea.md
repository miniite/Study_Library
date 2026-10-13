---
type: "practice"
difficulty: "Deep"
---

# A REST endpoint aggregates data from Postgres and Elasticsearch and the two drift apart during reindexing, so callers see inconsistent counts. How would you make the endpoint present a coherent view during reindex?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Consistent Pagination](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Consistent_Pagination.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve the issue of presenting a coherent count from a REST endpoint that aggregates data from both PostgreSQL and Elasticsearch (ES) during an index reindexing period, we need to ensure consistency in the counts returned by the API. The key challenge is handling the discrepancy between these two sources as they drift apart temporarily.

#### Triage
Understanding the Problem

The question asks how to make a REST endpoint present a coherent view of aggregated data despite differences arising during ES reindexing. This involves understanding the nature of the inconsistencies, identifying potential solutions, and ensuring that the application logic handles these discrepancies gracefully.

#### Assumptions
The PostgreSQL database is considered the authoritative source for the final state.
The Elasticsearch index is being updated in parallel with ongoing queries to the API.
The reindexing process can take a non-trivial amount of time, leading to temporary drifts between the two data sources.

#### First-Principles Solution

Combine Counts from Both Sources:

Fetch the latest count from PostgreSQL (authoritative source).
Fetch the current count from Elasticsearch (approximate but up-to-date).

Adjust Elasticsearch Count Based on Last Known State:

Use a timestamp to track when the last full sync between PostgreSQL and ES occurred.
Adjust the Elasticsearch count to account for any changes that have happened since the last sync.

Return a Combined View:

Present the combined counts in a way that reflects both sources' data.
Provide additional metadata (e.g., drift indicators) to help callers understand the current state.
Detailed Steps:

Fetch Counts from Both Databases:

SQL
-- PostgreSQL Query to get latest count
SELECT COUNT(*) FROM some_table;

-- Elasticsearch Query to get current count
GET /some_index/_count

Handle Reindexing Drift:

Use a timestamp stored in the application metadata or within ES to determine when the last full sync occurred.
PYTHON
def fetch_counts():
    postgres_count = execute_postgres_query("SELECT COUNT(*) FROM some_table")
    es_current_count = execute_es_query("/some_index/_count")

    # Fetch the last full-sync timestamp
    last_sync_timestamp = get_last_full_sync_timestamp()

    # Adjust ES count based on drift since last sync
    adjusted_es_count = adjust_es_count(es_current_count, last_sync_timestamp)

    return (postgres_count, adjusted_es_count)

def adjust_es_count(current_es_count, last_sync_timestamp):
    # Example logic to adjust the count; this would be specific to your use case
    if current_es_count > postgres_count:
        drift_count = current_es_count - postgres_count
        es_adjusted_count = max(0, current_es_count - (drift_count * 0.9))
    else:
        es_adjusted_count = current_es_count

    return es_adjusted_count

Return Combined and Adjusted Counts:

PYTHON
@app.route('/api/combined_counts')
def combined_counts():
    postgres_count, adjusted_es_count = fetch_counts()

    response = {
        "postgres_count": postgres_count,
        "es_adjusted_count": adjusted_es_count,
        "last_sync_timestamp": last_sync_timestamp,
        "drift_indicator": abs(postgres_count - adjusted_es_count)
    }

    return jsonify(response)

#### Closing Summary
and Common Mistakes/Follow-Ups:
Ensure Consistency: Always return a combined count that reflects the latest state, even during reindexing periods.
Provide Metadata: Include additional metadata like last_sync_timestamp and drift_indicator to help callers understand the current state.
Edge Cases: Consider edge cases such as sudden large spikes or drops in counts due to reindexing, and ensure your logic handles these gracefully.

#### Common mistakes
include:

Not accounting for the drift between PostgreSQL and Elasticsearch accurately.
Failing to provide sufficient metadata that could help callers understand the current state.
Ignoring performance implications of real-time adjustments, especially if the datasets are large.

**Code:**

```python
def fetch_counts():
    postgres_count = execute_postgres_query("SELECT COUNT(*) FROM some_table")
    es_current_count = execute_es_query("/some_index/_count")

    last_sync_timestamp = get_last_full_sync_timestamp()

    adjusted_es_count = adjust_es_count(es_current_count, last_sync_timestamp)

    return (postgres_count, adjusted_es_count)

def adjust_es_count(current_es_count, last_sync_timestamp):
    drift_count = current_es_count - postgres_count
    es_adjusted_count = max(0, current_es_count - (drift_count * 0.9))

    return es_adjusted_count

@app.route('/api/combined_counts')
def combined_counts():
    postgres_count, adjusted_es_count = fetch_counts()

    response = {
        "postgres_count": postgres_count,
        "es_adjusted_count": adjusted_es_count,
        "last_sync_timestamp": last_sync_timestamp,
        "drift_indicator": abs(postgres_count - adjusted_es_count)
    }

    return jsonify(response)
```

**Example input / output:**

```text
{
  "postgres_count": 123456,
  "es_adjusted_count": 120978,
  "last_sync_timestamp": 1628596753,
  "drift_indicator": 2478
}
```

**Flow:**

1. Yes
2. Yes
3. No
4. No
5. Fetch PostgreSQL Count
6. Is Last Full Sync TimestampKnown?
7. Fetch Elasticsearch CurrentCount
8. Adjust Elasticsearch CountBased on Drift
9. Combine Counts and ReturnResponse
10. Return Error or Retry FetchingES Count
11. Return API Response

