---
type: "practice"
difficulty: "Expert"
---

# Define RTO and RPO for the observability data estate itself. If the telemetry pipeline goes down during an outage, you're blind. Architect DR so the monitoring plane survives a region loss, and explain why observability DR differs from OLTP DR.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Centralized vs Mesh Observability Data](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Centralized_vs_Mesh_Observability_Data.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In defining the Recovery Time Objective (RTO) and Recovery Point Objective (RPO) for the observability data estate, we need to ensure that our monitoring plane remains resilient against regional outages while also considering how this differs from traditional operational transactions. This involves understanding the criticality of different components within the data architecture—whether it’s a data lake, warehouse, or mesh—and tailoring recovery strategies accordingly.

#### Triaging the Question

The question is essentially asking us to design disaster recovery (DR) mechanisms for the observability data estate, focusing on ensuring that monitoring and telemetry continue uninterrupted even when part of our infrastructure fails. The RTO specifies how quickly we need to recover from a failure, while the RPO indicates how much data loss can be tolerated.

#### Assumptions
We are dealing with an environment where multiple regions or zones are involved.
There is a critical time window during which telemetry must remain available (RTO).
Data integrity and accuracy are paramount for observability purposes.
The observability data estate includes both real-time and historical data.
Step-by-Step Solution

Identify Critical Components:

Data Ingestion Layer: This is where raw telemetry from various sources (e.g., sensors, logs) is collected.
Processing Layer: Aggregates and transforms data for ingestion into the data storage layers.
Storage Layers:
Data Lake: Stores raw telemetry data for long-term analysis.
Data Warehouse: Houses processed historical data used for business intelligence and analytics.
Mesh Network: A distributed architecture that allows for real-time querying of data across regions.

Define RTO and RPO for Each Component:

RTO for Data Ingestion Layer: Typically, this should be within seconds to minutes depending on the criticality of the system. For instance, if a region loss occurs, we need telemetry from other healthy regions or an immediate fallback to a cloud service.
RPO for Data Ingestion Layer: As raw data is immutable and essential, any RPO here would ideally be zero (no data loss). However, practical constraints might necessitate a non-zero value.

Data Storage Layers:

Data Lake:
RTO: Ensure data lake services are highly available with failover mechanisms in place.
RPO: Zero RPO for critical telemetry data; less stringent for non-critical logs, but still very low.
Data Warehouse:
RTO: High availability and quick recovery mechanisms like hot backups or replication to another region.
RPO: Again, zero RPO is ideal, but practical constraints might allow a small window of acceptable data loss.

Mesh Network:

RTO: Extremely low; immediate failover to a backup zone should be possible.
RPO: Zero RPO for critical queries and transactions; less stringent for non-critical ones.

DR Strategy Implementation:

Implement multi-region architectures with cross-zone replication.
Use cloud services that offer built-in DR features such as AWS S3, Redshift, or Google BigQuery.
Utilize Kubernetes or container orchestration tools to manage and ensure high availability of critical components.

#### Closing Summary

In summary, the RTO for the observability data estate should be set at extremely low levels, aiming for immediate recovery. The RPO will typically be zero, reflecting the need for minimal data loss in this context. Different layers within the data architecture require tailored DR strategies based on their specific roles and criticality.

#### Common Mistakes/Follow-Ups
Overlooking Data Ingestion Layer: Ensuring that real-time telemetry continues to flow is often overlooked.
Inadequate RPO for Historical Data: While high availability is crucial, too much focus might be placed on raw data at the expense of processed historical data.
Ignoring Edge Cases: Understanding how partial outages affect various components and ensuring comprehensive failover mechanisms.

**Comparison:**

```
Component	RTO (Minutes)	RPO (Seconds)
Data Lake	5	0
Data Warehouse	60	0
Mesh Network	30	0
```

**Code:**

```python
# Example Python code snippet for basic multi-region replication setup using AWS services
import boto3

def set_multi_region_replication(bucket_name, region):
    s3 = boto3.client('s3')
    # Set up cross-zone replication
    response = s3.put_bucket_replication(
        Bucket=bucket_name,
        ReplicationConfiguration={
            'Role': 'arn:aws:iam::123456789012:role/S3ReplicationRole',
            'Rules': [
                {
                    'Id': 'CrossRegionReplicationRule',
                    'Filter': {'Prefix': ''},
                    'TargetBucket': bucket_name,
                    'ReplicaKmsKeyId': 'alias/example-key-id',
                    'Status': 'Enabled',
                    'WithHttpOverS3Enabled': True
                }
            ]
        }
    )
    return response
```

**Example input / output:**

```text
Input:
- Telemetry data loss in one region.
Output:
- Immediate failover to another healthy region or cloud service for continued monitoring.
- Replication setup ensures minimal data loss and high availability across regions.
```

**Data:**

```json
{
  "DataLake": {
    "RTO": "5 minutes",
    "RPO": "0 seconds"
  },
  "DataWarehouse": {
    "RTO": "1 hour",
    "RPO": "0 seconds"
  },
  "MeshNetwork": {
    "RTO": "30 seconds",
    "RPO": "0 seconds"
  }
}
```

**Flow:**

1. Data Ingestion Layer
2. Replicate to Cloud Service
3. Failover to Another Region
4. Monitor and Alert
5. Sync with Main Storage
6. Ensure Continuity ofMonitoring

