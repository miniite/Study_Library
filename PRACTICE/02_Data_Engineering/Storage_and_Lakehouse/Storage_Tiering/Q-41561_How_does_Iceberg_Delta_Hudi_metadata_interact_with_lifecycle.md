---
type: "practice"
difficulty: "Expert"
---

# How does Iceberg / Delta / Hudi metadata interact with lifecycle transitions, and what breaks if lifecycle rules move data files out from under the table's manifest?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Storage Tiering Design](<../../../../02_Data_Engineering/Storage_and_Lakehouse/Storage_Tiering/Storage_Tiering_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with storage tiering and lifecycle management in big data platforms like Iceberg, Delta Lake, and Hudi, understanding how metadata interacts with file transitions is crucial to ensure the integrity of your data lake operations. This answer will explain how these systems handle lifecycle transitions and what potential issues arise if lifecycle rules move files out from under a table's manifest.

#### Triaging the Question

The question revolves around two main aspects:

Metadata Interaction: How does Iceberg, Delta Lake, or Hudi’s metadata interact with file transitions as part of lifecycle management?
Breaking Scenarios: What potential issues arise if lifecycle rules move files out from under a table's manifest?

#### Assumptions
We are dealing with an enterprise-grade data platform.
The focus is on Iceberg and Delta Lake, with some comparison to Hudi for completeness.

#### First-Principles Solution
Iceberg Metadata Interaction

Iceberg uses a manifest file system, which keeps track of the latest files in each partition. When a lifecycle rule moves files from Hot to Cold storage, the manifest is updated to reflect this change. However, if files are moved out of the manifest's purview (e.g., to Glacier), Iceberg may lose visibility over these files, leading to potential data inconsistencies.

Steps Involved:

Manifest Update: Lifecycles trigger updates in the manifest file, recording which files belong to each partition.
File Removal: If a lifecycle moves a file out of storage (e.g., from Hot to Glacier), it is no longer part of the active manifest.
Data Inconsistency: If the data engineer performs operations that expect these files to be in Hot storage, they will encounter missing or inconsistent data.
Delta Lake Metadata Interaction

Delta Lake uses a transaction log and metadata structure. Each transaction (write, read) is recorded in the log, which can include file movement events. Lifecycles move files from one tier to another but do not affect the transaction logs directly.

Steps Involved:

Transaction Log: Records each operation on the table.
File Movement: Moves files between tiers without altering the metadata structure.
Data Consistency: As long as transactions are replayed correctly, data consistency is maintained even if some files move to Cold or Glacier storage.
Hudi Metadata Interaction

Hudi (Huawei Delta) uses a combination of transaction logs and manifest files similar to Iceberg. However, it introduces mechanisms like compact and merge operations which can handle file movements more gracefully.

Steps Involved:

Transaction Logs: Records all transactions.
Compact Operations: Helps in merging small files into larger ones for better storage efficiency.
Manifest Files: Updated as new versions of data are written, but lifecycle rules may still cause inconsistencies if files are moved out of the manifest’s scope.

#### Common Mistakes
and Follow-Ups
Ignoring Manifest Changes: Failing to update the manifest when files move between tiers can lead to missing or stale data.
Not Validating Transactions: Assuming that transaction logs will handle all file movement scenarios without validating can result in operational issues.
Over-reliance on Metadata: Reliance solely on metadata structures (like manifests) for tracking data integrity may not be sufficient if files are moved out of scope.

#### Closing Summary

In summary, the interaction between storage tiering and lifecycle management systems like Iceberg, Delta Lake, and Hudi involves updating metadata to reflect file movements. However, moving files out from under a table's manifest can lead to data inconsistencies. Ensuring robust lifecycle policies that maintain visibility over all relevant files is crucial for maintaining data integrity.

#### Common Mistakes
Failing to Update Manifests: Not keeping the manifest up-to-date when files are moved between tiers.
Overlooking Transaction Logs: Assuming transaction logs alone will handle file movements without validation.
Ignoring Metadata Structures: Reliance solely on metadata structures can lead to inconsistencies if files are moved out of scope.

**Comparison:**

| File | Partition Key | Status   |
|------|---------------|----------|
| /path/to/hot_data/2023-10-01/data.parquet | date         | Hot      |
| /path/to/cold_data/2023-10-01/data.parquet | date         | Cold     |

**Code:**

```python
# Example of a lifecycle rule in Delta Lake
from delta import *
deltaTable = DeltaTable.forName(spark, "example_table")
lifecyclePolicy = {
    "rules": [
        {
            "name": "move_to_cold",
            "enabled": True,
            "prefix": "/path/to/hot_data/",
            "timeout": 14,
            "status": "active"
        }
    ]
}
deltaTable.updateLifecycles(lifecyclePolicy)
```

**Example input / output:**

```text
Input:
- Delta Lake table with lifecycle rule to move files older than 14 days to Cold storage.

Output:
- Manifest file updated.
- Files moved from Hot to Cold storage.
- Potential issues if files are moved out of the manifest's scope (e.g., to Glacier).
```

**Data:**

```json
{
    "manifestFile": {
        "partitionKey": "date",
        "files": [
            "/path/to/hot_data/2023-10-01/data.parquet",
            "/path/to/hot_data/2023-10-02/data.parquet"
        ]
    }
}
```

**Flow:**

1. File Write
2. Lifecycle Rule Triggers
3. Update Manifest
4. File Moved to Hot Storage
5. Transaction Log Updated
6. File Moves Out of Hot Storage
7. Manifest No Longer Tracks File
8. Data Inconsistency Possible

