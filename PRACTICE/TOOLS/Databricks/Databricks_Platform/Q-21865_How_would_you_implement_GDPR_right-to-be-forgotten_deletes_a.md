---
type: "practice"
difficulty: "Deep"
---

# How would you implement GDPR right-to-be-forgotten deletes across a Delta table that has time travel and 30 days of retained history?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Delta Live Tables and Serving](<../../../../TOOLS/Databricks/Databricks_Platform/Delta_Live_Tables_and_Serving.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the implementation of GDPR right-to-be-forgotten (RTBF) deletes in a Databricks environment with Delta tables and 30-day retention, we need to ensure that personal data can be promptly deleted from the live dataset while respecting historical data integrity. This involves a combination of technical processes and operational procedures.

Firstly, it's crucial to understand what RTBF means: the right for individuals to request the deletion of their personal data, which must be honored by organizations within 20 days from receiving the request. In our context, this implies that once a user requests their data be deleted, we need to remove their records not just in the live dataset but also from previous versions as retained by Delta's time travel feature.

To implement RTBF deletes effectively, follow these steps:

User Request Handling: Establish a secure and efficient mechanism for users or administrators to submit RTBF requests.
Data Identification: Use metadata about user data within the Delta table to identify which records need to be deleted.
Delete Operation: Execute a delete operation on both the live dataset and historical versions of the data.
Audit Logs: Maintain an audit log to track all RTBF actions for regulatory compliance.

Let's break down each step in detail:

User Request Handling

Implement a user-friendly interface or API where users can submit their request.
Ensure that requests are validated and logged, with timestamps and other relevant details.
Map the request to a specific user identifier (e.g., email address).

Data Identification

Use the user identifier from the RTBF request to query the Delta table for matching records.
Leverage Databricks' SQL capabilities or PySpark to write a query that identifies all records related to the user.

Delete Operation

For the live dataset, use the DELETE command in Spark SQL or PySpark:
SQL
-- Example using Spark SQL
DELETE FROM delta_table WHERE email = 'example@example.com';
To handle historical versions, we need to identify which historical snapshots contain the data to be deleted. This can be done by querying Delta's transaction log:
PYTHON
# Example in PySpark
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("RTBFDelete").getOrCreate()

def get_snapshot_ids(email, spark):
    df = spark.read.format('delta').load(delta_table_path)
    snapshot_ids = df.filter(df.email == email).selectExpr('snapshot_id').distinct().collect()
    return [record.snapshot_id for record in snapshot_ids]

# Apply the delete operation to historical snapshots
snapshot_ids_to_delete = get_snapshot_ids('example@example.com', spark)
for snapshot_id in snapshot_ids_to_delete:
    spark.sql(f"DELETE FROM delta_table FOR SYSTEM_TIME AS OF {snapshot_id} WHERE email = 'example@example.com'")

Audit Logs

Maintain an audit table to log all RTBF actions, including the request timestamp, user identifier, and action details.
SQL
CREATE OR REPLACE TABLE audit_log (
    id INT,
    user_email STRING,
    action TIMESTAMP,
    operation STRING,
    notes STRING
);

INSERT INTO audit_log (id, user_email, action, operation, notes)
VALUES (1, 'example@example.com', current_timestamp(), 'RTBF DELETE', 'User requested deletion of their data');

By following these steps, we can ensure that RTBF requests are handled efficiently and accurately in a Databricks environment with Delta tables. However, it’s important to consider the potential trade-offs and challenges:

Performance: Deleting large numbers of records from historical snapshots can impact performance.
Data Integrity: Ensuring that all relevant historical versions are deleted while maintaining data consistency is crucial.
Regulatory Compliance: Thoroughly documenting and auditing RTBF actions ensures compliance with GDPR requirements.

**Comparison:**

```
User Request
Validate and Log Request
Map Email to Record ID
(Delete Live Dataset)
Identify Historical Snapshots
Log Action in Audit Table
Delete from HistoricalSnapshots
Audit Log Entry Created
```

**Code:**

```

-- Example DELETE statement in Delta SQL DELETE FROM delta_table WHERE email = 'example@example.com';

CODE
Copy

```python
# Example PySpark code for historical snapshot deletion
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("RTBFDelete").getOrCreate()

def get_snapshot_ids(email, spark):
    df = spark.read.format('delta').load(delta_table_path)
    snapshot_ids = df.filter(df.email == email).selectExpr('snapshot_id').distinct().collect()
    return [record.snapshot_id for record in snapshot_ids]

# Apply the delete operation to historical snapshots
snapshot_ids_to_delete = get_snapshot_ids('example@example.com', spark)
for snapshot_id in snapshot_ids_to_delete:
    spark.sql(f"DELETE FROM delta_table FOR SYSTEM_TIME AS OF {snapshot_id} WHERE email = 'example@example.com'")
SQL
Copy
-- Example audit log table creation and insertion
CREATE OR REPLACE TABLE audit_log (
    id INT,
    user_email STRING,
    action TIMESTAMP,
    operation STRING,
    notes STRING
);

INSERT INTO audit_log (id, user_email, action, operation, notes)
VALUES (1, 'example@example.com', current_timestamp(), 'RTBF DELETE', 'User requested deletion of their data');
```

**Example input / output:**

```

Example input:

User requests deletion of email example@example.com in live and historical versions.

Output:

Live dataset deleted.
Historical snapshots with the user's data are identified and deleted.
Audit log entry created indicating RTBF action.
MARKDOWN
Copy
| Action ID | User Email   | Action Time  | Operation    | Notes                  |
|-----------|-------------|--------------|--------------|------------------------|
| 1         | example@example.com | 2023-10-05T14:30:00.000Z | RTBF DELETE | User requested deletion of their data |
```

**Data:**

```yaml
delta_table_path: /path/to/delta/table
```

**Flow:**

1. Validate and Log Request
2. Identify Records in LiveTable
3. Identify HistoricalSnapshots
4. Delete from HistoricalSnapshots
5. Log Audit
6. User Request
7. Map Email to Record ID
8. (Delete from Delta Table)
9. D
10. E
11. Audit Log Entry Created

