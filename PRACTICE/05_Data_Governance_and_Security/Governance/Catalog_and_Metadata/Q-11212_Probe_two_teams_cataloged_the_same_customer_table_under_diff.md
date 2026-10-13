---
type: "practice"
difficulty: "Expert"
---

# Probe: two teams cataloged 'the same' customer table under different URNs, so lineage forks and search returns both. How do you design entity dedup/merge without corrupting downstream references?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Scaling Lineage Graphs](<../../../../05_Data_Governance_and_Security/Governance/Catalog_and_Metadata/Scaling_Lineage_Graphs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When two or more teams independently catalog the same data entities in a data catalog (e.g., customer tables with different URNs), it can lead to duplicated metadata entries and lineage forks, complicating search and potentially causing downstream issues. To design an effective entity deduplication and merge strategy without corrupting downstream references, we need to carefully navigate through several considerations: data integrity, lineage preservation, and operational efficiency.

#### Triaging the Question

The core of this question revolves around ensuring that metadata duplication is resolved while maintaining a clear lineage for all downstream processes. The primary challenges include:

Identifying Duplicates: Determining which entries are truly duplicates.
Resolving Merges: Deciding how to consolidate duplicated metadata.
Preserving Lineage: Ensuring that the deduplication process does not break existing data relationships.

#### Assumptions
We have a robust lineage graph representing all data dependencies.
The catalog system supports versioning and soft-deletes for metadata entries.
Metadata includes relevant attributes like URN, schema, and lineage information.
Downstream systems are designed to handle changes in metadata references gracefully.

#### First-Principles Solution

Identify Duplicates:

Use a fingerprint or hash of the metadata (e.g., customer table schema) as a unique identifier.
Compare fingerprints across different URNs to identify potential duplicates.

Merge Strategy:

Soft Delete: Mark duplicate entries as "soft deleted" rather than fully removing them to preserve lineage and auditability.

#### Cons
olidate Metadata: For each duplicated entry, decide on the primary metadata source (e.g., by team ID or timestamp).
Update Lineage Graph: Update all references in the lineage graph to point to the chosen primary metadata.

Preserve Lineage:

Maintain a detailed audit log of all changes.
Ensure that downstream systems can trace back to original metadata sources if needed.

Operational Considerations:

Batch processing: Deduplication and merge should be done in batch processes to minimize disruption.
Incremental Updates: Gradually update the catalog to avoid a sudden influx of changed entries.
Testing: Rigorously test the deduplication logic with realistic scenarios.

#### Closing Summary

To effectively manage duplicated metadata, we need a systematic approach that combines accurate identification, strategic merging, and meticulous lineage preservation. By using versioning, soft-deletion, and detailed audit logs, we can ensure data integrity while maintaining clear lineage for downstream systems.

#### Common Mistakes/Follow-Ups
Ignoring Lineage: Not maintaining the correct lineage can break critical dependencies.
Overzealous Deletion: Fully deleting metadata without proper auditing or versioning can lead to loss of historical context.
Synchronization Issues: Ensuring that all downstream systems are updated consistently is crucial.

Would you like me to elaborate on any specific aspect, such as implementation details or a flowchart?

**Comparison:**

```
Metadata ID	URN	Schema	Status	Timestamp
1	urn1	customer	primary	2023-10-01T10:00Z
2	urn2	customer	soft_deleted	2023-10-01T09:00Z
```

**Code:**

```python
# Example code snippet for soft delete and deduplication logic

def deduplicate_and_merge(metadata):
    fingerprints = {}
    # Step 1: Identify duplicates
    for entry in metadata:
        fingerprint = hash(entry)
        if fingerprint not in fingerprints:
            fingerprints[fingerprint] = [entry]
        else:
            fingerprints[fingerprint].append(entry)

    # Step 2: Resolve merges
    for fingerprint, entries in fingerprints.items():
        if len(entries) > 1:
            primary_entry = max(entries, key=lambda e: e['timestamp'])
            for entry in entries:
                if entry != primary_entry:
                    entry['status'] = 'soft_deleted'
            metadata.remove(primary_entry)
```

**Example input / output:**

```text
Input:
[
    {
        "URN": "urn1",
        "schema": "customer",
        "metadata": {"id": 1, "timestamp": "2023-10-01T09:00Z"}
    },
    {
        "URN": "urn2",
        "schema": "customer",
        "metadata": {"id": 1, "timestamp": "2023-10-01T10:00Z"}
    }
]

Output:
[
    {
        "URN": "urn1",
        "schema": "customer",
        "metadata": {"id": 1, "timestamp": "2023-10-01T10:00Z", "status": "primary"}
    },
    {
        "URN": "urn2",
        "schema": "customer",
        "metadata": {"id": 1, "timestamp": "2023-10-01T09:00Z", "status": "soft_deleted"}
    }
]
```

**Data:**

```json
[
    {
        "URN": "urn1",
        "schema": "customer",
        "metadata": {"id": 1, "timestamp": "2023-10-01T09:00Z"}
    },
    {
        "URN": "urn2",
        "schema": "customer",
        "metadata": {"id": 1, "timestamp": "2023-10-01T10:00Z"}
    }
]
```

**Flow:**

1. Yes
2. No
3. Identify Duplicates
4. Are fingerprints identical?
5. Merge Duplicates
6. Mark as Soft Deleted
7. Update Metadata
8. Preserve Lineage

