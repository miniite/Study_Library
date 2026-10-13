---
type: "practice"
difficulty: "Expert"
---

# How do you design lineage that survives a pipeline that reads from S3, transforms in Databricks, and lands in BigQuery — where does OpenLineage instrumentation break across cloud boundaries?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Cross-cloud Data Contracts](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Multi_Cloud_Data/Cross-cloud_Data_Contracts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design lineage for a multi-cloud data architecture involving pipelines that read from S3, transform in Databricks, and land in BigQuery, we need to ensure that lineage information is captured and preserved across these different cloud environments and tools. OpenLineage is a powerful tool for this purpose, but its implementation requires careful consideration of how metadata is instrumented at each step of the pipeline.

#### Triaging the Question

The core issue here is ensuring that lineage data is properly propagated through S3 (AWS), Databricks (AWS/Azure/Google Cloud), and BigQuery (GCP). OpenLineage can provide this by capturing details such as the source, target, and transformations applied. However, since these are different cloud services, we need to ensure consistent metadata capture and reporting.

#### First-Principles Solution
Identify Metadata Requirements: Determine what metadata is necessary for lineage, including job IDs, steps, inputs, outputs, and transformations.
Instrumentation in Databricks: Use OpenLineage's APIs or libraries to capture transformation details within Databricks jobs.
Bridge Between Clouds: Implement a bridge service that collects metadata from different cloud environments and consolidates it into a unified lineage store.

#### Cons
istent Metadata Storage: Store the lineage data centrally, ensuring it can be queried across clouds.
Detailed Steps

Metadata Collection in Databricks:

Utilize OpenLineage's Python client to capture metadata during job execution.
Define clear schemas for the metadata required (e.g., source files, transformations, target tables).

Bridge Service:

Develop a bridge service that can connect to different cloud APIs (S3, BigQuery) and collect lineage data.
Use standardized formats like JSON or Protocol Buffers to ensure interoperability.

#### Cons
istent Storage:

Store the collected metadata in a central database or a cloud storage service accessible from all environments.
Implement a schema registry to manage versioning and evolution of metadata schemas.

Monitoring and Validation:

Set up monitoring to validate that metadata is correctly captured at each step.
Regularly review lineage reports to ensure accuracy and completeness.

#### Common Mistakes
and Follow-Ups
Ignoring Consistency: Failing to maintain consistent metadata across different clouds can lead to fragmented lineage data.
Overlooking Edge Cases: Not accounting for edge cases such as failed jobs, partial transformations, or transient errors in S3 access.
Lack of Automation: Relying too heavily on manual processes for collecting and reporting lineage.
Scalability Issues: Failing to consider the scalability of the bridge service when dealing with large volumes of data.

#### Example
Workflow
Data Read from S3
Databricks Job
Metadata Captured byOpenLineage Client
Bridge Service CollectsMetadata
Unified Lineage Store
Transformed Data Written toBigQuery
Databricks Job
Metadata Captured byOpenLineage Client
Bridge Service CollectsMetadata
Unified Lineage Store

#### Closing Summary

Designing lineage in a multi-cloud environment requires careful orchestration of metadata collection and reporting. By leveraging tools like OpenLineage and implementing a bridge service, we can ensure that lineage data is consistent and complete across different cloud services.

Self-Check

This answer covers all aspects of the question and addresses potential pitfalls.

**Comparison:**

```
Transformation	Description
filter_rows	Filter out rows with invalid data.
join_tables	Join this table with another for analysis.
```

**Code:**

```python
# Example Databricks Job Metadata Capture
from openlineage.client import Client

def capture_metadata(job_id, inputs, outputs, transformations):
    client = Client(url="http://openlineage.server/api/v1")
    job_details = {
        "jobId": job_id,
        "inputs": inputs,
        "outputs": outputs,
        "transformations": transformations
    }
    response = client.emit_run(job_details)
```

**Example input / output:**

```text
Input: S3 file path: s3://data-bucket/input.csv
Metadata Captured:
{
  "source_files": ["s3://data-bucket/input.csv"],
  "transformations": [
    {
      "name": "filter_rows",
      "description": "Filter out rows with invalid data"
    }
  ],
  "target_table": "bigquery://my-project/dataset/table"
}
Output: Bridge service collects and stores this metadata.
```

**Data:**

```json
{
  "source_files": ["s3://bucket/path/to/file.csv"],
  "transformations": [
    {
      "name": "filter_rows",
      "description": "Filter out rows with invalid data"
    },
    {
      "name": "join_tables",
      "description": "Join this table with another for analysis"
    }
  ],
  "target_table": "bigquery://project/dataset/table"
}
```

**Flow:**

1. Data Read from S3
2. Databricks Job
3. Metadata Captured byOpenLineage Client
4. Bridge Service CollectsMetadata
5. Unified Lineage Store
6. Transformed Data Written toBigQuery
7. Databricks Job
8. Metadata Captured byOpenLineage Client
9. Bridge Service CollectsMetadata
10. Unified Lineage Store

