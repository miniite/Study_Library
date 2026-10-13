---
type: "practice"
difficulty: "Basic"
---

# What is the difference between full refresh and incremental sync in data ingestion?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Incremental Sync and Connector Migration](<../../../../02_Data_Engineering/Pipelines_and_Integration/Data_Ingestion/Incremental_Sync_and_Connector_Migration.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Full refresh and incremental sync are two fundamental strategies used in data ingestion to determine how new or updated data should be handled. A full refresh involves re-ingesting all the data, typically from a source like a database table, which can be time-consuming if the dataset is large. Incremental sync, on the other hand, only ingests newly added or changed records, making it more efficient for ongoing data integration tasks.

To understand these concepts better, let's break down their key differences and how they are implemented in tools like Airbyte/Fivetran:

Full Refresh:

A full refresh involves completely re-ingesting all the data from a source.
This can be useful when you need to start with a fresh dataset or when the source system is updated frequently, making incremental sync impractical.
In terms of implementation in Airbyte/Fivetran, it means that the entire table or schema will be read and processed again.

Incremental Sync:

Incremental sync focuses on ingesting only new records or changes to existing records.
This is achieved by tracking a timestamp field (or other unique identifiers) that indicates when data was last updated.
In Airbyte/Fivetran, this could involve using incremental lookback windows, WHERE clauses in SQL queries, or specific schema mappings.

Now let's see how these concepts can be implemented:

#### Implementation
Considerations

Full Refresh:

#### Pros
Ensures consistency and accuracy of the data.

#### Cons
Time-consuming for large datasets and may cause temporary disruptions if the source system is unavailable during the full refresh.

Incremental Sync:

#### Pros
More efficient, especially with ongoing data streams; reduces storage and processing costs.

#### Cons
Requires additional logic to track and process changes; can be complex to implement correctly.

#### Example Scenario
s

Let's consider a scenario where we are ingesting customer records from a database:

Full Refresh:

SQL
INSERT INTO destination_table (customer_id, name, email)
SELECT * FROM source_table;

Incremental Sync: Assuming the last_updated_at field in the source table tracks when each record was last modified:

SQL
INSERT INTO destination_table (customer_id, name, email)
SELECT * FROM source_table WHERE last_updated_at > (SELECT MAX(last_updated_at) FROM destination_table);

#### Common Mistakes
and Follow-Ups
Incorrect Incremental Logic: Forgetting to update the timestamp field in the source or destination table can lead to incorrect data being ingested.
Data Consistency Issues: Full refresh might not be suitable if the source system is frequently updated, as it could cause inconsistencies.

In conclusion, understanding and choosing between full refresh and incremental sync depends on the specific requirements of your data ingestion pipeline. Full refresh ensures consistency but can be resource-intensive, while incremental sync is more efficient but requires careful implementation to avoid issues.

