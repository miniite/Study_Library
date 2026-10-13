---
type: "practice"
difficulty: "Deep"
---

# Your Iceberg reads got slow over weeks even though data volume is flat. How do you investigate manifest bloat and what maintenance procedures fix it?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Table Format Internals](<../../../../02_Data_Engineering/Storage_and_Lakehouse/Table_Formats/Table_Format_Internals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with an Iceberg table that has been slowing down over time, despite a stable data volume, one of the primary suspects is "manifest bloat." This issue arises because of the way Iceberg handles metadata updates and compaction, leading to an accumulation of old manifests. To investigate and mitigate this problem, we need to understand how manifest files are managed in Iceberg and then implement appropriate maintenance procedures.

**Code:**

```python
# Example Python code snippet for querying Iceberg tables (using PyArrow)
import pyarrow as pa

def check_manifests(table_location):
    client = pyarrow.parquet.ParquetDataset(table_location).client
    manifests = client.list_files("manifest-*")
    print(f"Number of manifest files: {len(manifests)}")
```

**Example input / output:**

```text
# Example input and output for checking manifest files
Input:
check_manifests("/path/to/iceberg/table")

Output:
Number of manifest files: 500

This indicates a potential issue with excessive manifest bloat.
```

**Data:**

```text
- `table_location`: The path to the Iceberg table.
```

**Flow:**

1. Yes
2. No
3. Start
4. Is the number of manifest filessignificantly higher thanexpected?
5. Maintain manifests bycompaction or vacuum
6. Continue monitoringperformance
7. Perform compaction to reducemanifest bloat
8. Verify performanceimprovement after compaction
9. If necessary, run vacuumoperation for cleanup

