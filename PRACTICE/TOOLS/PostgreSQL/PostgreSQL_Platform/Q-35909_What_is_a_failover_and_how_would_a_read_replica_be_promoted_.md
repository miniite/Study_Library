---
type: "practice"
difficulty: "Expert"
---

# What is a failover and how would a read replica be promoted to primary in a managed setup like RDS or Aurora?

**Difficulty:** Expert · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Replica Lag and Read Your Writes](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Replica_Lag_and_Read_Your_Writes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a managed database service like Amazon RDS or Amazon Aurora, a failover is a process where a secondary instance (read replica) takes over as the primary instance when the current primary becomes unavailable or needs maintenance. This transition ensures minimal downtime and high availability for your application.

#### Triage

The question is asking about how to promote a read replica to primary in a managed database setup like RDS or Aurora, which involves understanding the failover process, the underlying architecture, and the steps required to ensure smooth operation after promotion.

#### Assumptions
The primary instance has failed or requires maintenance.
The read replica is fully up-to-date with the primary.
Both instances are managed by a service like RDS or Aurora, which provides automated failover capabilities.

#### First-Principles Solution
Check Replica Status: Ensure that the read replica is in a healthy state and has caught up to the latest data from the primary.
Promotion via Management Console/API: Use the AWS management console or API to promote the replica to primary.
Failover Process: During promotion, the service performs a failover to switch the roles of the instances.
Detailed Steps:
Health Check:
Verify that the read replica is functioning correctly by checking its status and lag from the primary instance.
Promotion via Management Console/CLI:
In AWS RDS, navigate to the RDS console, select the read replica, and initiate a failover operation.
Alternatively, use the AWS CLI command aws rds failover-db-instance.
Post-Failover Verification:
Once promoted, ensure that the new primary is healthy and accessible.
Check connectivity and performance metrics.

#### Closing Summary

Promoting a read replica to primary in a managed database service like RDS or Aurora involves ensuring the health of the replica, initiating a failover via management tools or APIs, and verifying the new primary's status. This process ensures minimal downtime and high availability for your application.

#### Common Mistakes
Failing to check the replica’s lag before promoting it.
Not verifying the health and connectivity of the new primary after promotion.
Ignoring potential data inconsistencies if the failover happens during a large transaction or when there's significant write activity.
Depth + Language Matching

For an expert, understanding the nuances of failover and the specific steps involved in managing read replicas is crucial. This includes considering edge cases such as during peak load times or during significant data writes, ensuring proper configuration to minimize risks, and leveraging automation features provided by AWS services like Aurora Read Replicas.

**Code:**

```bash
# Example of promoting a replica via AWS CLI
aws rds failover-db-instance --db-instance-identifier my-replica
```

**Flow:**

1. Check Replica Status
2. Verify Health and Lag
3. Promote Replica viaManagement Console/API
4. Post-Failover Verification
5. Health Check New Primary

