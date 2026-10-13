---
type: "practice"
difficulty: "Expert"
---

# You must design the disaster-recovery story for a stateful system with a 15-minute RTO and near-zero RPO across regions. Reconcile that with the physics of cross-region replication latency, and describe how you'd continuously test failover so the RTO is real, plus how you handle failback afterward.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Sync vs Async Replication](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Sync_vs_Async_Replication.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a disaster-recovery strategy for a stateful system with a 15-minute Recovery Time Objective (RTO) and near-zero Recovery Point Objective (RPO) across regions, we need to reconcile the inherent latency challenges of cross-region replication with robust failover testing. The key is to ensure that our failover process is not just fast but also reliable, and that failback can be smoothly managed.

#### Triage
Understanding the Question

The question revolves around designing a disaster-recovery system for a stateful application that operates across multiple AWS regions. The primary constraints are an RTO of 15 minutes and a near-zero RPO. This means we need to ensure that in the event of a failure, data can be quickly and accurately restored from a replica located in another region, while minimizing downtime.

Key Assumptions
Latency: Cross-region replication latency is typically around 20-150ms.
Bandwidth: Adequate bandwidth between regions for fast data transfer.
Storage: Sufficient storage capacity and IOPS (Input/Output Operations Per Second) in the replica region.

#### Cons
istency Models: Strong consistency across writes, but eventual consistency for reads to reduce latency.
Step-by-Step Solution

Replication Strategy

Use asynchronous replication with eventual consistency for high availability and low latency.
Implement synchronous replication (write-ahead logs) to ensure data integrity during failover scenarios.

High Availability Design

Deploy the application in two or more AWS regions, with each region having a primary and secondary replica.
Utilize AWS Global Accelerator for consistent low-latency access to the closest available instance.
Implement an automatic failover mechanism that detects application failures and switches traffic to the secondary replica.

Replication Latency Handling

Use cross-region replication via AWS DMS (Database Migration Service) or CloudEndure for database state synchronization.
Employ DynamoDB Streams with Global Tables for real-time data replication across regions.

RPO Management

Ensure that the secondary replica is updated within 15 minutes of any write operation.
Implement a periodic checkpointing mechanism to minimize RPO during network partitions or outages.

Failover Testing

Conduct regular failover tests by manually triggering the failover process and verifying data consistency across regions.
Use AWS CloudFormation for automated testing scripts that simulate region failures and validate recovery procedures.

Failback Handling

Implement a robust health check mechanism to determine when it’s safe to switch back from the secondary replica.
Develop a manual or semi-automated failback process to transition primary roles once the issue is resolved.

#### Closing Summary

By leveraging AWS services like DMS, CloudEndure, and Global Tables, we can achieve near-zero RPO while ensuring an RTO of 15 minutes. Regular testing and validation ensure that our disaster-recovery plan remains reliable and effective.

#### Common Mistakes/Follow-Ups
Mistake: Over-reliance on synchronous replication, which can lead to high latency.

#### Follow-Up
Discuss the trade-offs between consistency models and performance in different use cases.

By focusing on robust design, continuous testing, and failback management, we ensure a disaster-recovery system that meets both RTO and RPO requirements efficiently.

**Code:**

```python
# Example Python script for failover detection using boto3 (AWS SDK)
import boto3

def check_region_health():
    ec2 = boto3.client('ec2')
    response = ec2.describe_regions()
    unhealthy_regions = [region['RegionName'] for region in response['Regions']
                         if not is_region_available(region['RegionName'])]
    return unhealthy_regions

def is_region_available(region):
    # Placeholder function to check availability
    pass

def trigger_failover():
    # Logic to switch roles and update DNS records using Route 53
    pass
```

**Example input / output:**

```

Input:

Manually trigger failover for the application in us-west-2.

Output:

Traffic is redirected to the secondary region eu-central-1.
Data consistency checks are performed.
Health checks indicate that the secondary instance is ready.

Request/Response Example:

JSON
Copy
{
  "status": "Failover Initiated",
  "new_primary_region": "eu-central-1"
}
```

**Data:**

```json
{
  "regions": ["us-west-2", "eu-central-1"],
  "primary_region": "us-west-2",
  "secondary_region": "eu-central-1"
}
```

