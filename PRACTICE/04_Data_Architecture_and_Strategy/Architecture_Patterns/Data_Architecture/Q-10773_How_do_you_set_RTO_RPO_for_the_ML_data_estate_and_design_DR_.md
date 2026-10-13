---
type: "practice"
difficulty: "Expert"
---

# How do you set RTO/RPO for the ML data estate and design DR so a region loss doesn't take out feature serving for a fraud model? Why is DR for online feature stores fundamentally harder than for the offline lake?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Data Product SLAs](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Data_Product_SLAs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When setting Recovery Time Objective (RTO) and Recovery Point Objective (RPO) for an ML data estate, we need to balance between data freshness, system availability, cost, and risk tolerance. The goal is to ensure that feature serving remains uninterrupted even when a region experiences a failure. This involves designing robust disaster recovery (DR) strategies specifically tailored to the nature of online feature stores compared to offline data lakes.

To start with, let’s break down what RTO and RPO mean in this context:

RPO is how much historical data loss we are willing to accept.
RTO is how quickly the system must recover from a failure.

In an ML data estate, these objectives depend on factors like the criticality of feature updates, regulatory requirements, and business needs. For instance, in fraud detection models, any delay or data loss can significantly impact model accuracy and real-time decision-making capabilities.

To design effective DR strategies, we need to understand the specific architecture and operational patterns of our ML data estate:

Feature Serving Layer: This is critical for real-time decisions and must be highly available.
Offline Data Lake: Stores historical data used for training models and offline analytics.
Mesh Architecture: A networked structure that can route traffic to multiple availability zones or regions.

Now, let’s delve into the detailed solution:

RTO and RPO Calculation

Identify Critical Components:

Feature Serving Layer: High availability is crucial as this layer directly impacts real-time decisions.
Offline Data Lake: While important for training models, it can have some data loss.

RPO for Feature Serving Layer:

Criticality of Data Freshness: For fraud detection, RPO might be set to zero or a very short duration (e.g., every 5 minutes) to ensure minimal historical data loss.
Backup Strategy: Implement real-time backups using streaming technologies like Kinesis Firehose for critical feature updates.

RTO for Feature Serving Layer:

Recovery Time Requirements: If the system must remain fully functional, RTO might be set to a few minutes (e.g., 5-10 minutes) to failover to another region or availability zone.
Redundant Geographical Locations: Use multi-region setups with read replicas and active-active configurations.

RPO for Offline Data Lake:

Backup Strategy: For offline data, RPO can be more relaxed (e.g., every hour) as the impact of a short-term data loss is lower.
Archival Mechanisms: Implement periodic archival to S3 or other durable storage solutions.

DR Plan for Offline Data Lake:

Failover Strategy: Use cloud services like AWS DMS or Azure Data Factory for automated failover and recovery.
Data Consistency Checks: Regularly run consistency checks to ensure data integrity post-failover.
Designing DR Strategies

Multi-Region Setup:

Deploy feature serving layer in multiple regions (e.g., US-East, US-West) with active-active or active-passive configurations.
Use a load balancer to route traffic based on the availability of each region.

Disaster Recovery for Feature Serving Layer:

Implement cross-region replication and failover mechanisms using services like AWS Route 53 failover routing policies.
Test the failover process regularly to ensure smooth transition in case of a failure.

Data Archival Mechanisms:

Set up automated archival processes to move older data from the main storage layer to an archive storage (e.g., S3 Glacier).
Ensure that archived data can be quickly restored when needed.

Monitoring and Alerting:

Implement comprehensive monitoring and alerting for both online feature stores and offline data lakes.
Use tools like AWS CloudWatch or Azure Monitor to track RPO and RTO compliance.
Why DR for Online Feature Stores is Harder
Real-Time Constraints: Online systems require near-zero latency, which makes recovery from a failure more challenging.
Data Consistency: Ensuring data consistency across multiple regions can be complex due to network latency and transactional dependencies.
User Impact: Real-time services have strict SLAs (Service Level Agreements) that must be adhered to, making RTO a higher priority.

#### Common Mistakes
Ignoring Edge Cases: Failing to consider scenarios like network outages or prolonged downtime.
Overlooking Data Integrity: Not implementing robust data validation and consistency checks during failovers.
Inadequate Testing: Skipping regular testing of DR plans, leading to unexpected failures in critical moments.

#### Closing Summary

By carefully balancing RTO and RPO, implementing a multi-region setup with active-active or active-passive configurations, and using cloud services for automated recovery, we can ensure that the ML data estate remains robust against regional losses. The DR strategy for online feature stores is inherently more challenging due to real-time constraints and user impact, which necessitates a higher level of redundancy and consistency.

**Comparison:**

```
Component	RPO (minutes)	RTO (minutes)
Feature Serving Layer	5	10
Offline Data Lake	60	-
```

**Code:**

```python
# Example Python script for setting up cross-region replication in AWS S3
import boto3

def setup_cross_region_replication(bucket_name, region):
    s3 = boto3.client('s3')
    response = s3.put_bucket_replication(
        Bucket=bucket_name,
        ReplicationConfiguration={
            'Role': 'arn:aws:iam::123456789012:role/S3ReplicationRole',
            'Rules': [
                {
                    'ID': 'CrossRegionReplicationRule',
                    'Prefix': '',
                    'Status': 'Enabled',
                    'TargetBucket': f'arn:aws:s3:::{bucket_name}-{region}',
                    'TargetObjectKeyPrefix': '',
                    'StorageClass': 'STANDARD'
                }
            ]
        }
    )
    print(response)
```

**Example input / output:**

```text
Input:
- RPO for feature serving layer: every 5 minutes
- RTO for feature serving layer: 5-10 minutes
- Critical regions: US-East, US-West

Output:
- Setup cross-region replication between US-East and US-West using S3.
- Implement real-time backups with Kinesis Firehose for critical updates.
- Regularly test failover processes to ensure smooth transition in case of a failure.
```

**Data:**

```json
{
  "region": "us-east-1",
  "bucket_name": "fraud-detection-data-lake",
  "replication_role_arn": "arn:aws:iam::123456789012:role/S3ReplicationRole"
}
```

**Flow:**

1. Feature Serving Layer
2. US-East
3. US-West
4. Cross-Region Replication
5. Real-Time Backups
6. Kinesis Firehose
7. Regular Failover Testing
8. Smooth Transition in Case ofFailure

