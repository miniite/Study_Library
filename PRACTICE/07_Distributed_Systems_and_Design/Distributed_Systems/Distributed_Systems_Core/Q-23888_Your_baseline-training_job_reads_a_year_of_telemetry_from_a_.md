---
type: "practice"
difficulty: "Intermediate"
---

# Your baseline-training job reads a year of telemetry from a warehouse via a query that returns slightly different rows run-to-run, making baselines non-reproducible. Explain how snapshot isolation or dataset versioning fixes this and why immutable telemetry snapshots matter for AIOps audits.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Distributed Backfills](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Clock_Skew_and_Time_Windows/Distributed_Backfills.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In distributed systems, ensuring reproducibility in baseline-training jobs is crucial, especially when dealing with large-scale telemetry data that may change slightly over time. Snapshot isolation and dataset versioning are two techniques that address this issue by preserving a consistent state of the dataset at the point of snapshot creation. Immutable telemetry snapshots ensure that historical data remains unchanged, facilitating accurate AIOps audits.

#### Triaging the Question

The core of this question is about ensuring reproducibility in distributed systems where baseline-training jobs rely on large datasets like telemetry data. The challenge here is that the query returns slightly different rows each time it’s run, leading to non-reproducible baselines. We need to understand how snapshot isolation and dataset versioning can fix this issue.

#### First-Principles Solution

Snapshot Isolation:

Definition: Snapshot isolation allows multiple transactions to operate on a "view" of the database as it was at the start of their transaction, rather than a view that may have been updated by other concurrent transactions.

#### Implementation
By using snapshot isolation, we can create a consistent snapshot of the telemetry data at a specific point in time. This snapshot is then used for training models without being affected by changes made to the live dataset during the query execution.

Dataset Versioning:

Definition: Dataset versioning involves tagging and storing multiple versions of the same dataset, each representing a different state or point in time.

#### Implementation
Each snapshot created can be tagged with a unique identifier (version number) and stored separately. This ensures that we always have access to the exact data used during training at any given time.

Immutable Telemetry Snapshots:

Definition: Immutable snapshots mean that once a snapshot is created, it cannot be modified or deleted.

#### Implementation
By making these snapshots immutable, we guarantee their integrity and availability for future reference. This is particularly important in AIOps audits where the exact state of data at a specific time must be verifiable.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, snapshot isolation ensures that each baseline-training job operates on a consistent view of the telemetry data by creating a snapshot at the start of the transaction. Dataset versioning and immutable snapshots provide a means to store these consistent views for future reference, ensuring reproducibility and auditability.

#### Common Mistakes/Follow-Ups

Failing to Understand Snapshot Granularity: One common mistake is assuming that snapshot isolation works only at the database level when it can be applied to individual transactions.
Neglecting Data Integrity: Not implementing immutability ensures that snapshots can be altered, leading to potential inconsistencies in audits.
Ignoring Historical Context: In AIOps, maintaining historical context of data changes is crucial. Without proper versioning and snapshot management, it becomes difficult to track the evolution of models over time.

**Comparison:**

| Timestamp       | Value |
|-----------------|-------|
| 2023-10-01T00:00:01Z | 5.2   |
| 2023-10-01T00:00:02Z | 6.3   |

**Code:**

```python
# Example Python code for creating a snapshot isolation environment
from sqlalchemy import create_engine, func

engine = create_engine('postgresql://user:password@localhost/db')
with engine.connect() as connection:
    # Start a transaction with snapshot isolation
    connection.execute("SET TRANSACTION ISOLATION LEVEL SNAPSHOT")

    # Query telemetry data and save the results to a temporary table
    query = "CREATE TEMP TABLE telemetry_snapshot AS SELECT * FROM telemetry WHERE timestamp >= '2023-10-01' AND timestamp < '2024-10-01'"
    connection.execute(query)
```

**Example input / output:**

```plaintext
Input:
Query to fetch telemetry data for the year 2023.

Output:
[{"timestamp": "2023-10-01T00:00:01Z", "value": 5.2, "id": 1},
 {"timestamp": "2023-10-01T00:00:02Z", "value": 6.3, "id": 2}]
```

**Data:**

```json
{
  "telemetry_data": [
    {"timestamp": "2023-10-01T00:00:01Z", "value": 5.2, "id": 1},
    {"timestamp": "2023-10-01T00:00:02Z", "value": 6.3, "id": 2}
  ]
}
```

**Flow:**

1. Start
2. Create Snapshot
3. Query and Process Data
4. Store as Immutable Snapshot
5. Audit and Validate
6. End
7. ---

