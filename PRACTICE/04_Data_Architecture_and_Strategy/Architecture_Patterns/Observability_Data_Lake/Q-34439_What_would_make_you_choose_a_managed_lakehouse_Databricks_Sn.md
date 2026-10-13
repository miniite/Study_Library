---
type: "practice"
difficulty: "Deep"
---

# What would make you choose a managed lakehouse (Databricks/Snowflake) versus rolling your own S3-plus-Iceberg-plus-Trino stack for observability?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Telemetry Lake File Layout](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Observability_Data_Lake/Telemetry_Lake_File_Layout.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deciding between a managed lakehouse like Databricks or Snowflake and rolling your own S3-plus-Iceberg-plus-Trino stack for observability, several factors come into play, including ease of use, cost, performance, and maintenance. A managed lakehouse offers streamlined data management, robust features, and lower operational overhead compared to a self-managed setup. However, the latter provides more flexibility and control, which can be beneficial in specific scenarios.

**Comparison:**

```
Feature	Managed Lakehouse (Databricks/Snowflake)	Self-managed S3+Iceberg+Trino
Ease of Use	High	Moderate
Cost	Lower due to integrated billing	Higher, requires careful cost tracking
Performance	Optimized for query performance	Requires tuning and monitoring
Maintenance	Minimal, managed services	High, manual setup and maintenance
Flexibility	Limited	High
Security	Integrated with cloud security	Custom implementation
```

**Code:**

```python
# Example Trino query for Iceberg table
query = """
CREATE TABLE IF NOT EXISTS my_observable_data (
    timestamp TIMESTAMP,
    user_id BIGINT,
    event_type STRING
) WITH (
    format='DELTA',
    location='s3://my-bucket/iceberg-table'
)
"""
```

**Example input / output:**

```
Input: CREATE TABLE IF NOT EXISTS my_observable_data (timestamp TIMESTAMP, user_id BIGINT, event_type STRING) WITH (format='DELTA', location='s3://my-bucket/iceberg-table')
Output:
CODE
Copy
Table created successfully.
```

**Data:**

```json
{
  "bucket": "s3://my-bucket/",
  "table_location": "s3://my-bucket/iceberg-table",
  "data_lakehouse": {
    "provider": "Databricks/Snowflake"
  },
  "self_managed": {
    "storage": "S3",
    "table_format": "Iceberg",
    "query_engine": "Trino"
  }
}
```

**Flow:**

1. Managed Lakehouse
2. Streamlined Data Management
3. Lower Operational Overhead
4. Integrated BI and ML Tools
5. Self-managedS3+Iceberg+Trino
6. Manual Setup andMaintenance
7. Higher Cost with Custom Costs
8. Tuning Required forPerformance

