---
type: "practice"
difficulty: "Deep"
---

# How do you architect for reproducibility — 'give me exactly the dataset as it was on 2024-11-01 for this ML model'? What does time-travel/snapshotting cost you in storage and metadata, and where do you cap retention?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Reproducible Training Data Versions](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/Reproducible_Training_Data_Versions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To architect a system that provides reproducibility—specifically, "give me exactly the dataset as it was on 2024-11-01"—we need to design a robust data architecture capable of maintaining historical snapshots or time-travel capabilities. This involves understanding both the storage costs and metadata management overhead associated with snapshotting and deciding on an optimal retention period.

#### Triaging the Question

The core question is about ensuring that we can retrieve exactly the state of our dataset as it existed on a specific date (2024-11-01) for machine learning (ML) model training. This requires us to implement some form of time-travel or snapshotting mechanism. The interviewer will likely be probing your understanding of data architecture principles, storage management strategies, and cost considerations.

#### Assumptions
Data Sources: Assume the dataset comes from multiple sources like databases, logs, and external APIs.
Snapshotting Mechanism: We use a robust snapshotting system that can reliably capture and store historical states of the dataset.
Cost Considerations: Both storage costs for snapshots and metadata overhead will be significant factors.

#### First-Principles Solution

Data Lakes & Warehouses:

Data Lake: Use a data lake (e.g., AWS S3, Azure Data Lake Storage) to store all raw and semi-processed data.
Data Warehouse: Utilize a data warehouse (e.g., Snowflake, Google BigQuery) for transformed and consolidated datasets.

Snapshotting Strategy:

Implement an incremental snapshot strategy where only the changes since the last snapshot are stored.
Use a versioning system to label each snapshot with its exact timestamp or date identifier.

Metadata Management:

Maintain detailed metadata about each snapshot, including creation time and associated dataset state.
Ensure that metadata is easily searchable and accessible for quick retrieval of relevant snapshots.

RetentionPolicy:

Determine a reasonable retention period based on the business requirements and cost constraints.
Cap retention to minimize storage costs while ensuring compliance with data regulatory needs.

#### Implementation
Steps

Data Capture:

Implement real-time or batch processing pipelines that write changes to the data lake.
Use a timestamped naming convention for snapshots (e.g., dataset-2024-11-01-09:00).

Incremental Snapshotting:

Capture only incremental changes since the last snapshot.
This reduces storage overhead and improves efficiency.

Metadata Storage:

Use a database (e.g., PostgreSQL, Cassandra) to store metadata about each snapshot.
Include fields like snapshot_id, creation_time, dataset_state, etc.

RetentionPolicy Enforcement:

Implement automated policies that delete snapshots older than the retention period.
Monitor storage costs and adjust the retention policy as necessary.

#### Closing Summary

In summary, achieving reproducibility in a dataset involves implementing an incremental snapshotting strategy in combination with metadata management to track and retrieve historical states. The key is balancing storage efficiency with cost constraints while ensuring compliance with data retention policies.

#### Common Mistakes/Follow-Ups
Overhead of Snapshotting: Be cautious about the overhead introduced by frequent snapshots, especially in high-volume environments.
Metadata Management: Ensure robust metadata management to avoid difficulties in retrieving specific snapshots.
RetentionPolicy Adjustments: Regularly review and adjust retention policies based on storage costs and regulatory requirements.
Triaging Follow-Ups

The interviewer might ask:

Storage Cost Considerations:
How do you estimate the cost of storing historical data?
Impact on Performance:
What are the potential performance impacts of maintaining multiple snapshots?
Compliance Requirements:
How do regulatory requirements affect your retention strategy?

**Comparison:**

| Snapshot ID       | Creation Time     | Data Source | Last Updated            |
|-------------------|------------------|-------------|-------------------------|
| dataset-2024-11-01-09:00 | 2024-11-01T09:00:00Z | database    | 2024-11-01T08:59:00Z   |
| dataset-2024-11-01-10:00 | 2024-11-01T10:00:00Z | database    | 2024-11-01T09:59:00Z   |

**Code:**

```python
# Example Python code for snapshot metadata management

class SnapshotMetadata:
    def __init__(self, snapshot_id, creation_time, dataset_state):
        self.snapshot_id = snapshot_id
        self.creation_time = creation_time
        self.dataset_state = dataset_state

# Example function to create a snapshot and log its metadata
def create_snapshot(snapshot_id: str, creation_time: str, dataset_state: dict) -> SnapshotMetadata:
    metadata = SnapshotMetadata(
        snapshot_id=snapshot_id,
        creation_time=creation_time,
        dataset_state=dataset_state
    )
    # Log or store the metadata in a database
    return metadata

# Example function to retrieve a specific snapshot by ID
def get_snapshot_by_id(snapshot_id: str, metadata_store: list) -> SnapshotMetadata:
    for meta in metadata_store:
        if meta.snapshot_id == snapshot_id:
            return meta
    raise ValueError(f"Snapshot with ID {snapshot_id} not found")
```

**Example input / output:**

```text
Input:
- Request for snapshot with ID 'dataset-2024-11-01-09:00'

Output:
{
  "snapshot_id": "dataset-2024-11-01-09:00",
  "creation_time": "2024-11-01T09:00:00Z",
  "dataset_state": {
    "data_source": "database",
    "last_updated": "2024-11-01T08:59:00Z"
  }
}
```

**Data:**

```json
{
  "metadata_store": [
    {
      "snapshot_id": "dataset-2024-11-01-09:00",
      "creation_time": "2024-11-01T09:00:00Z",
      "dataset_state": {
        "data_source": "database",
        "last_updated": "2024-11-01T08:59:00Z"
      }
    },
    {
      "snapshot_id": "dataset-2024-11-01-10:00",
      "creation_time": "2024-11-01T10:00:00Z",
      "dataset_state": {
        "data_source": "database",
        "last_updated": "2024-11-01T09:59:00Z"
      }
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Data Source
4. Snapshot Creation?
5. Create Incremental Snapshot
6. Store Metadata in DB
7. Return Snapshot ID and State
8. Use Existing Snapshot

