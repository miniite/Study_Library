---
type: "practice"
difficulty: "Intermediate"
---

# What is AWS DMS and how can it be used for CDC?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ AWS DMS vs Debezium](<../../../../02_Data_Engineering/Pipelines_and_Integration/Change_Data_Capture/AWS_DMS_vs_Debezium.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

AWS Database Migration Service (DMS) is a fully managed service that facilitates the movement of data between different databases, including both on-premises and cloud-based databases. In the context of Change Data Capture (CDC), AWS DMS can be used to track changes in real-time or near-real-time within source databases, and replicate those changes to target databases, ensuring data consistency across multiple systems.

The key steps involve setting up an endpoint for your source database, configuring a replication instance, creating a task that defines the schema mappings, and specifying the CDC settings. Let's break down how DMS can be utilized effectively for CDC from first principles.

**Comparison:**

| Task Configuration | Description |
|-------------------|------------|
| ReplicationTaskIdentifier | Unique identifier for the replication task |
| SourceEndpointArn  | ARN of the source endpoint (e.g., RDS, Aurora) |
| TargetEndpointArn  | ARN of the target endpoint (e.g., Redshift, S3) |
| ReplicationInstanceArn | ARN of the DMS replication instance |
| TableMappings      | Rules defining how to map tables and columns between sources and targets |
| CDCStartTime       | Timestamp indicating when data capture should begin |
| MigrationType      | Type of migration (e.g., cdc-only, full-load) |

**Code:**

```python
# Example Python snippet to configure a basic AWS DMS task
import boto3

dms_client = boto3.client('dms')

response = dms_client.create_replication_task(
    ReplicationTaskIdentifier='cdc-task-123',
    SourceEndpointArn='arn:aws:dms:region:account-id:endpoint-type/source-endpoint-arn',
    TargetEndpointArn='arn:aws:dms:region:account-id:endpoint-type/target-endpoint-arn',
    ReplicationInstanceArn='arn:aws:dms:region:account-id:replication-instance-arn',
    TableMappings='{"rules": [{"rule-type":"selection","rule-id":"1","rule-name":"Rule 1","apply-rules-to-source":"true", "column-selection":["*"]}] }',
    CDCStartTime="2023-09-05T00:00:00Z",
    MigrationType='cdc-only'
)
print(response)
```

**Example input / output:**

```text
# Example input and output for creating a CDC task using AWS DMS API

# Input (as JSON)
input_data = {
    "ReplicationTaskIdentifier": "cdc-task-123",
    "SourceEndpointArn": "arn:aws:dms:region:account-id:endpoint-type/source-endpoint-arn",
    "TargetEndpointArn": "arn:aws:dms:region:account-id:endpoint-type/target-endpoint-arn",
    "ReplicationInstanceArn": "arn:aws:dms:region:account-id:replication-instance-arn",
    "TableMappings": "{\"rules\": [{\"rule-type\":\"selection\",\"rule-id\":\"1\",\"rule-name\":\"Rule 1\",\"apply-rules-to-source\":\"true\", \"column-selection\":[\"*\"]}] }",
    "CDCStartTime": "2023-09-05T00:00:00Z",
    "MigrationType": "cdc-only"
}

# Output (as JSON)
output_data = {
    "ReplicationTask": {
        "ReplicationTaskIdentifier": "cdc-task-123",
        "SourceEndpointArn": "arn:aws:dms:region:account-id:endpoint-type/source-endpoint-arn",
        "TargetEndpointArn": "arn:aws:dms:region:account-id:endpoint-type/target-endpoint-arn",
        "ReplicationInstanceArn": "arn:aws:dms:region:account-id:replication-instance-arn",
        "TableMappings": "{\"rules\": [{\"rule-type\":\"selection\",\"rule-id\":\"1\",\"rule-name\":\"Rule 1\",\"apply-rules-to-source\":\"true\", \"column-selection\":[\"*\"]}] }",
        "CDCStartTime": "2023-09-05T00:00:00Z",
        "MigrationType": "cdc-only"
    }
}
```

**Data:**

```json
{
  "ReplicationTask": {
    "ReplicationTaskIdentifier": "cdc-task-123",
    "SourceEndpointArn": "arn:aws:dms:region:account-id:endpoint-type/source-endpoint-arn",
    "TargetEndpointArn": "arn:aws:dms:region:account-id:endpoint-type/target-endpoint-arn",
    "ReplicationInstanceArn": "arn:aws:dms:region:account-id:replication-instance-arn",
    "TableMappings": "{\"rules\": [{\"rule-type\":\"selection\",\"rule-id\":\"1\",\"rule-name\":\"Rule 1\",\"apply-rules-to-source\":\"true\", \"column-selection\":[\"*\"]}] }",
    "CDCStartTime": "2023-09-05T00:00:00Z",
    "MigrationType": "cdc-only"
  }
}
```

**Flow:**

1. Set up Source Endpoint
2. Create Replication Instance
3. Configure Target Endpoint
4. Define Table Mappings
5. Specify CDC Settings
6. Start Replication Task

