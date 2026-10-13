---
type: "practice"
difficulty: "Expert"
---

# How do you handle the case where re-embedding the whole corpus for an index migration takes six hours and can't block the deploy; walk me through dual-index, backfill, and read-switch phases.

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ Expand and Contract Migrations](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Idempotent_Deploy_Jobs/Expand_and_Contract_Migrations.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the scenario where re-embedding a large dataset (corpus) for an index migration takes six hours and cannot be blocked during deployment, we can implement a dual-index strategy with backfilling and read-switch phases. This approach ensures that the system remains operational throughout the migration without blocking critical deployments.

#### Triaging the Question

The core of this question revolves around ensuring minimal downtime during large-scale data migrations in a CI/CD pipeline. Specifically, it probes our understanding of how to manage incremental updates to an index while maintaining service availability. The key assumptions are:

The system must remain operational throughout the migration.
Re-embedding the entire corpus is too time-consuming and cannot be blocked.
Dual-Index Strategy

The dual-index strategy involves running a parallel index that gets updated incrementally, allowing ongoing reads from both indexes until the new index is fully ready. Here’s how it works:

Initial Setup: Have two indices: Current and New. The Current index contains the latest data.
Incremental Updates to New Index: As new data comes in or existing data needs updating, it gets added to both the Current and New indexes simultaneously (or nearly so).
Read-Only Phase: Reads are directed only to the Current index until the New index is fully populated.
Switch Phase: Once the New index is ready, all reads are switched over to it.
Backfilling

Backfilling ensures that the new index catches up with any data that was not updated during its initial creation phase. This step is crucial because the initial population of a large dataset can be time-consuming and might block other operations.

Backfill Process: A backfill process runs in parallel, updating the New index to ensure it includes all necessary embeddings.
Overlap: During this period, both indexes coexist, ensuring no data loss or staleness during the migration.
Read-Switch Phase

The final phase involves switching over reads entirely from the old (Current) index to the new (New) index.

Switching Mechanism: A controlled switch ensures that all read operations are redirected to the New index.
Validation: Verify that the New index is fully operational and matches the expected state before completing the switch.
Step-by-Step Solution

Initial Index Population:

Create an initial Current index with existing data embeddings.
Begin backfilling into a new, parallel New index as data updates occur.

Incremental Updates and Backfill:

For each incoming update or batch of data, apply the changes to both indexes simultaneously.
Ensure that the backfill process is efficient and doesn't stall deployment pipelines.

Read-Only Phase:

During this phase, all reads are directed to the Current index to maintain service availability.
Monitor the progress of the backfilling process.

Switch to New Index:

Once the New index is fully populated and validated, gradually switch over read operations.
Ensure that no data inconsistencies occur during this transition phase.

#### Closing Summary

This dual-index approach ensures minimal downtime by allowing the system to continue operating with an up-to-date dataset while migrating a new version in parallel. The backfilling process ensures that the new index catches up with any missed updates, and the read-switch mechanism provides a seamless transition without service interruptions.

#### Common Mistakes/Follow-Ups
Blocking Deployment: Not properly handling the migration could lead to blocking deployments.
Data Inconsistency: Failing to validate the New index before switching can result in inconsistent data.
Performance Impact: Overlooking the performance impact of maintaining two indexes simultaneously.

**Comparison:**

```
Phase	Description
Initial Setup	Create Current and New indexes, populate Current.
Incremental Updates	Simultaneously update both Current and New as new data arrives.
Backfill Process	Ensure New index is fully populated with existing missing updates.
Read-Only Phase	Direct all reads to Current, ensuring service availability.
Switch Phase	Gradually switch read operations to New after validation.
```

**Code:**

```python
def migrate_index(current_index, new_index):
    # Incremental updates and backfilling logic
    for batch in get_data_batches():
        update_current_and_new(batch)

    validate_new_index(new_index)

def update_current_and_new(batch):
    current_index.update(batch)
    new_index.backfill(batch)

def validate_new_index(new_index):
    assert new_index.is_fully_populated()
```

**Example input / output:**

```text
Input:
- Current index with existing embeddings and data
- New index initialized but empty

Output:
- Dual-index setup: both `Current` and `New` indexes updated incrementally
- Backfill process ensuring new index catches up with missing updates
- Smooth transition to fully using the `New` index for reads
```

**Data:**

```json
{
  "CurrentIndex": {
    "embeddings": ["embedding1", "embedding2"],
    "data": ["data1", "data2"]
  },
  "NewIndex": {
    "embeddings": [],
    "data": []
  }
}
```

**Flow:**

1. Initial Setup
2. Incremental Updates & Backfill
3. Read-Only Phase
4. Switch Phase

