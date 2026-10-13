---
type: "practice"
difficulty: "Deep"
---

# Your incremental sync missed updates because the source app does in-place updates without touching updated_at. How do you make ingestion trustworthy?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Incremental Sync and Connector Migration](<../../../../02_Data_Engineering/Pipelines_and_Integration/Data_Ingestion/Incremental_Sync_and_Connector_Migration.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that data is ingested accurately and up-to-date despite the source application performing in-place updates, we need to rethink our approach beyond traditional incremental sync methods. Specifically, we'll leverage a combination of logical clocks, custom-built extractors, and possibly alternative sync strategies like full refreshes with deduplication or watermarking. The core idea is to ensure that all changes are captured even if the updated_at timestamp doesn't reflect the actual data change.

This question tests your understanding of the limitations of incremental sync in modern ETL pipelines and your ability to design robust, reliable solutions that can handle edge cases like those presented by source applications with sophisticated update mechanisms. The key assumptions include understanding the data schema, the nature of the updates performed, and the requirements for ensuring data integrity.

Let's break down the solution step-by-step:

Understand the Problem:

Traditional incremental sync relies on tracking changes via a updated_at timestamp or similar mechanism.
The source app performs in-place updates without changing this timestamp, making it appear that no new data has been added.

Identify Alternative Methods:

Logical Clocks: Use unique IDs for rows and a logical clock to track changes. This ensures that every change is uniquely identifiable, even if the updated_at field doesn't change.
Custom Extractors: Develop custom extractors to detect when data has been updated in place but hasn't touched the timestamp. These can use heuristic or semantic checks based on the schema.

Implement Deduplication:

For full refreshes, implement robust deduplication logic using unique keys and timestamps to ensure that only recent changes are ingested.
Use a staging area to merge data from different sources before loading into the final destination.

Watermarking:

Introduce watermark fields in your schema to track when updates occur. This can be done by creating a last_updated field or similar metadata that captures changes more reliably than timestamps.

Hybrid Approach:

Combine incremental sync with full refreshes based on the logical clock. For example, if a row hasn't changed significantly over a period of time, perform an incremental update; otherwise, do a full refresh to ensure all recent changes are captured.

Validation and Monitoring:

Implement validation checks post-ingestion to ensure data integrity.
Use monitoring tools to track the effectiveness of your ingestion strategy and make adjustments as needed.

**Comparison:**

```
Logical Clock	Source ID	Updated Fields
12345	987	price, description
```

**Code:**

```python
# Example custom extractor logic in Python using Airbyte/Python SDK
from airbyte_protocol.models import SyncMode, StreamSlice

def extract_in_place_updates(stream_slice: StreamSlice):
    # Custom logic to identify rows that have been updated in place but not by changing the timestamp
    for record in stream_slice.records:
        if is_record_updated(record):  # Custom function to detect updates based on schema
            yield record

def is_record_updated(record) -> bool:
    # Implement specific checks based on the data model and business logic
    return check_semantic_changes(record)
```

**Example input / output:**

```text
Input: Record with updated 'description' but no change in 'updated_at'
Output: Extractor detects the update and yields a record for ingestion.
```

**Data:**

```json
{
  "source": {
    "id": 123,
    "name": "Product",
    "attributes": [
      {"key": "price", "value": 99.99},
      {"key": "description", "value": "Updated description"}
    ]
  },
  "logical_clock": 456,
  "last_updated": "2023-10-01T12:00:00Z"
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Is Record Updated?
5. Extract Record
6. Continue Processing
7. Update Logical Clock
8. Ingest Data into Staging Area
9. Merge and Deduplicate inFinal Destination

