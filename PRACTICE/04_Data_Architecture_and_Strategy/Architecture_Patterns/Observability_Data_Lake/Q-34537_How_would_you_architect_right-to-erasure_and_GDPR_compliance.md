---
type: "practice"
difficulty: "Expert"
---

# How would you architect right-to-erasure and GDPR compliance on an immutable append-only lake without a rewrite storm every time a deletion request lands?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Exactly Once into Observability Lakes](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Observability_Data_Lake/Exactly_Once_into_Observability_Lakes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To architect right-to-erasure (RTE) and GDPR compliance on an immutable, append-only data lake, we need to design a system that can handle deletions without causing a rewrite storm or violating data immutability principles. The core idea is to maintain a layer of metadata tracking deletions and use selective ingestion to ensure only the latest valid state is served. Here’s how you could approach this:

#### Triaging the Question

The question asks for an architecture that allows for right-to-erasure (RTE) in an immutable append-only data lake, ensuring GDPR compliance while avoiding unnecessary data rewrites. The key aspects being probed are:

Understanding of GDPR and RTE: GDPR mandates the ability to delete personal data at the request of the user.
Handling Immutability Constraints: Data lakes are typically append-only, meaning they cannot be altered once written.
Avoiding Rewrite Storms: Minimizing the number of updates required when a deletion is requested.

#### First-Principles Solution

Metadata Layer for Deletions:

Maintain a separate metadata layer that tracks deletions.
Each delete operation logs an entry in this metadata, indicating which records should be ignored or treated as non-existent.

Selective Ingestion and Query Processing:

When a query is executed against the data lake, use the metadata to filter out deleted entries before serving results.
Implement selective ingestion pipelines that only process and load valid (non-deleted) records into downstream systems.

Data Versioning and Segmentation:

Use versioned storage to keep multiple snapshots of your data over time.
When a delete request is made, update the metadata rather than rewriting old data, thereby maintaining immutability principles.
Step-by-Step Solution
Create a Metadata Store: This store will track all deletion requests and ensure they are atomic and durable.
Implement a Query Filter Mechanism: When queries come in, use this metadata to filter out any deleted records before processing them further.
Selective Ingestion Pipelines: Set up pipelines that can selectively ingest data from the main lake based on current metadata statuses.

#### Closing Summary

In summary, architecting RTE and GDPR compliance involves creating a robust metadata layer for tracking deletions and implementing selective ingestion to avoid unnecessary rewrites. This approach ensures that the append-only nature of the data lake is preserved while meeting legal requirements.

#### Common Mistakes/Follow-Ups
Failing to Use Metadata Properly: Not leveraging a separate metadata store can lead to incorrect deletion tracking, causing unintended exposure or retention.
Overlooking Data Versioning: Ignoring versioning might result in incomplete or inconsistent data states when deletions are applied.
Underestimating Query Filtering Complexity: Complex queries may require intricate logic to filter out deleted records efficiently.

**Comparison:**

```
Metadata Entry ID	Operation	Timestamp	Record ID	Table Name
1	delete	2023-10-05T14:00:00Z	12345	customer_data
2	delete	2023-10-06T15:00:00Z	12346	customer_data
```

**Code:**

```python
# Example metadata entry for a deletion request
deletion_entry = {
    "operation": "delete",
    "timestamp": "2023-10-05T14:00:00Z",
    "record_id": 12345,
    "table_name": "customer_data"
}
```

**Example input / output:**

```text
Input:
- A query to retrieve customer data for a specific time period.

Output:
- The system consults the deletion metadata and filters out any records that have been marked as deleted.
- Only valid, non-deleted records are returned in the result set.
```

**Data:**

```json
{
    "deletion_log": [
        {
            "operation": "delete",
            "timestamp": "2023-10-05T14:00:00Z",
            "record_id": 12345,
            "table_name": "customer_data"
        },
        {
            "operation": "delete",
            "timestamp": "2023-10-06T15:00:00Z",
            "record_id": 12346,
            "table_name": "customer_data"
        }
    ]
}
```

**Flow:**

1. Yes
2. No
3. Query Received
4. Is Record Deleted?
5. FILTER OUT RECORD
6. Include Record in Result
7. Update Metadata withDeletion
8. Return Filtered Results

