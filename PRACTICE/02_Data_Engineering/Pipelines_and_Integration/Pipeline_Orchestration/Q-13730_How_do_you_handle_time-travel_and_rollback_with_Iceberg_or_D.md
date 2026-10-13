---
type: "practice"
difficulty: "Deep"
---

# How do you handle time-travel and rollback with Iceberg or Delta when a bad pipeline run corrupted the gold table that BI is already querying?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ File and Table Formats for Features](<../../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/File_and_Table_Formats_for_Features.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with a scenario where a bad pipeline run has corrupted a "gold" table used by Business Intelligence (BI) queries, handling time-travel and rollback using Iceberg or Delta Lake becomes crucial. The core idea is to leverage these data lake technologies' built-in features for managing historical versions of the data. This ensures that BI can continue to query from a consistent version of the dataset without being affected by recent errors.

First, let's triage the question: We are dealing with a critical issue where a pipeline failure has introduced corruption into the gold table. The interviewer is probing our understanding of how to manage such situations, including data lineage tracking, versioning, and rollback strategies. Key assumptions include the availability of historical versions within Iceberg or Delta Lake tables.

To solve this from first principles:

Understanding Data Versioning: Both Iceberg and Delta Lake maintain a history of changes through their transaction logs. These technologies allow you to query specific versions of data based on timestamp or commit ID, ensuring that BI queries can be redirected to non-corrupted historical versions.

Detecting and Isolating the Corrupt Data:

Audit Logs: Review pipeline audit logs to identify when the corruption occurred.
Data Validation Checks: Implement robust validation checks in your pipelines to catch errors before they reach production tables.

Rollback Strategy:

Incremental Rollbacks: For minor issues, you might revert changes incrementally using transaction IDs or timestamps.
Full Table Recovery: In severe cases, restore the table from a known good backup or point-in-time recovery.

Using Iceberg/Delta Lake Features:

Time Travel Queries: Both Iceberg and Delta support time travel queries (SELECT ... AS OF 'timestamp' for Delta) to query historical data.
Snapshots and Watermarks: Track snapshots of your data over time and use watermarks to manage incremental updates.

Implementing a Fail-Safe Pipeline:

Retry Mechanisms: Ensure that your pipeline has robust retry mechanisms in place to handle transient errors.
Error Logging: Log detailed error messages for each processing step, including the cause of failure.

Testing and Validation:

Regularly validate the data using checksums or other integrity checks.
Test rollback procedures by simulating failures in a non-production environment before deploying them to production.

#### Closing summary
Handling time-travel and rollback effectively requires leveraging features inherent to Iceberg and Delta Lake, such as versioning and transaction logs. By implementing robust validation, logging, and rollback strategies, you can ensure that BI queries always have access to clean, consistent data despite pipeline failures. Common mistakes include neglecting thorough testing or failing to maintain historical backups.

**Comparison:**

| Commit ID | Timestamp       | Action              |
|-----------|-----------------|--------------------|
| 89f352b7  | 2023-10-01T14:00 | Insert             |
| 9f3a82f7  | 2023-10-02T10:00 | Revert to snapshot |

**Code:**

```python
# Example of using Delta Lake's time travel query in Spark
df = spark.read.format("delta").option("timestampAsOf", "2023-10-01T14:00:00Z").load("/path/to/table")
```

**Example input / output:**

```

Input: A bad pipeline run caused corruption in the gold table. Output: BI queries are redirected to a known good snapshot from 2023-10-01, ensuring consistent data for analysis.

Request:

CODE
Copy
revert /path/to/table AS OF '2023-10-02T10:00:00Z'
Response:

Table has been successfully reverted to the specified snapshot.
```

**Data:**

```json
{
  "table_version": {
    "id": "89f352b7-cf1d-4c7b-b64a-f2e4c8e3d542",
    "timestamp": "2023-10-01T14:00:00Z"
  },
  "rollback_snapshot": {
    "id": "9f3a82f7-cf6b-4c6b-b64e-f2e5c9e3d542",
    "timestamp": "2023-10-02T10:00:00Z"
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Detect Corruption
6. Is it a minor issue?
7. Revert Incrementally
8. Have historical backups?
9. Restore from Backup
10. Full Table Recovery

