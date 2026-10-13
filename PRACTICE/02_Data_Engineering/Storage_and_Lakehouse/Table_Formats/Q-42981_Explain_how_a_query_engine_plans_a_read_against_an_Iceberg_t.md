---
type: "practice"
difficulty: "Deep"
---

# Explain how a query engine plans a read against an Iceberg table: catalog lookup → snapshot → manifest list → manifest filtering → data-file scan. Where can each step become a bottleneck?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Table Format Internals](<../../../../02_Data_Engineering/Storage_and_Lakehouse/Table_Formats/Table_Format_Internals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To explain how a query engine plans a read operation against an Iceberg table, we need to understand its internal workflow and potential bottlenecks at each stage:

#### Overview
TL;DR: The process involves several steps, including catalog lookup, snapshot identification, manifest list filtering, and finally the data-file scan. Each step can introduce performance issues due to various factors.

#### Triage
The query engine's read planning for an Iceberg table follows a structured approach:

Catalog Lookup: Determines the location of the metadata.
Snapshot Identification: Finds the correct version of the metadata.
Manifest List Filtering: Filters out unused or irrelevant manifests to find relevant files.
Data-File Scan: Reads and processes the actual data.

Step-by-Step Solution:

Catalog Lookup:

The query engine starts by looking up the catalog entry for the Iceberg table, typically stored in a metadata store like HDFS or another file system.
Key metadata such as the table's UUID and location of snapshots are retrieved.
This step can be slow if the catalog is not well-optimized or if there are frequent lookups.

Snapshot Identification:

The engine then identifies the correct snapshot based on versioning information in the catalog entry.
Snapshots capture the state of the table at different points in time, with each snapshot containing lineage metadata and partitioning details.
Bottleneck potential: If there are many snapshots, this step can take considerable time.

Manifest List Filtering:

Each snapshot has a manifest list that contains pointers to data files.
The query engine filters the manifest list based on the read criteria (e.g., partitions, timestamps).
This filtering process can be slow if there are too many manifests or if the partitioning is not well-defined.

Data-File Scan:

After identifying relevant manifests, the query engine performs a scan of the data files.
These files are usually stored in Parquet or another columnar format for efficient querying.
Bottleneck potential: If the file size and number are large, this step can be time-consuming.

#### Closing Summary
+ Common Mistakes/Follow-Ups:

#### Common Mistakes
Overlooking the importance of snapshot management, not optimizing partitioning, and failing to properly index metadata.

#### Follow-Ups
How can we optimize each step? What are the trade-offs between performance and storage cost?

**Comparison:**

| Step | Description |
|------|-------------|
| 1    | Catalog Lookup - Retrieve catalog entry for the Iceberg table |
| 2    | Snapshot Identification - Identify correct version of metadata |
| 3    | Manifest List Filtering - Filter manifests based on read criteria |
| 4    | Data-File Scan - Read and process data files |

**Code:**

```python
# Example pseudo-code for a simplified read planning process
def plan_read(table_name):
    catalog = get_catalog_entry(table_name)
    snapshot_uuid = find_correct_snapshot(catalog)
    manifest_list = fetch_manifest_list(snapshot_uuid)
    filtered_manifests = filter_relevant_manifests(manifest_list, query_criteria)
    data_files = scan_data_files(filtered_manifests)
```

**Example input / output:**

```text
Input: Query to read data from Iceberg table 'sales' with timestamp >= 2023-01-01

Output:
1. Catalog Lookup: UUID = 12345678-90ab-cdef-1234-567890abcdef, Snapshots = [version 1 (2023-01-01), version 2 (2023-02-01)]
2. Snapshot Identification: Correct Snapshot UUID = 12345678-90ab-cdef-1234-567890abcdef
3. Manifest List Filtering: Relevant Manifests = [manifest_1, manifest_2]
4. Data-File Scan: Read data_files = [file1.parquet, file2.parquet]
```

**Data:**

```json
{
  "catalog_entry": {
    "uuid": "12345678-90ab-cdef-1234-567890abcdef",
    "snapshots": [
      {"version": 1, "timestamp": "2023-01-01T00:00:00Z"},
      {"version": 2, "timestamp": "2023-02-01T00:00:00Z"}
    ]
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Yes
6. No
7. Catalog Lookup
8. Is Catalog Optimized?
9. Snapshot Identification
10. Optimize Catalog
11. Are Snapshots Well-Defined?
12. Manifest List Filtering
13. Re-evaluate Partitioning
14. Are Manifests FilteredEfficiently?
15. Data-File Scan
16. Optimize Manifest Filtering
17. Optimize Catalog Lookup
18. Improve Partitioning
19. Optimize Manifest Filtering

