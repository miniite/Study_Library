---
type: "practice"
difficulty: "Expert"
---

# How would you decide your RTO and RPO for a B2B SaaS, and then design backups and failover to actually meet those numbers within budget?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ SLOs and Error Budgets](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Availability_Zone_Resilience/SLOs_and_Error_Budgets.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To determine Recovery Time Objective (RTO) and Recovery Point Objective (RPO) for a B2B SaaS application, we need to consider several factors including the criticality of data, service uptime requirements, regulatory compliance, customer impact, and budget constraints. The RTO is how quickly a system must be operational after a failure, while RPO defines how far back in time the state of the system can regress due to an outage.

For this B2B SaaS, let's assume it operates 24/7 with frequent data modifications, and customers expect near-zero downtime. We will first triage these requirements:

RTO: Since customer satisfaction is high-stakes, a RTO of less than 5 minutes might be desirable.
RPO: Given the nature of B2B SaaS, where transactions are critical, an RPO of no more than 30 seconds could be necessary.

Next, we will design the backup and failover strategy to meet these objectives within budget:

Backup Strategy:

Use cloud-based object storage for backups.
Implement incremental backups every 5 minutes during business hours.
Full backups are taken once per day at night to ensure a complete restore point.

Failover Strategy:

Set up an automated failover mechanism in another AWS region, using CloudFormation or Terraform templates.
Use Route 53 for DNS health checks and failover routing.
Configure RDS or database replication to replicate data in real-time to the secondary region.

Testing & Validation:

Perform regular failover tests monthly to validate the process.
Monitor backup integrity and performance using CloudWatch and SNS notifications.

To meet these requirements within budget, we must optimize resources:

Utilize AWS Backup for automated backups and cost optimization.
Leverage AWS Database Migration Service (DMS) for real-time data replication between regions.
Use Amazon Route 53 health checks to minimize DNS propagation delays during failover.

Here’s a step-by-step plan:

Assess current RTO and RPO gaps with the desired objectives.
Define the disaster recovery plan, including backup frequency and storage.
Set up automated failover in another AWS region using CloudFormation templates.
Implement continuous monitoring and testing to ensure the solution is reliable.

**Comparison:**

```
Resource	Action	Frequency
Database Backup	Incremental	Every 5 mins
	Full	Daily
DNS Health Check	Automated failover	Real-time
```

**Code:**

```python
# Example Python script for setting up a basic CloudFormation template for failover
import boto3

def create_cloudformation_stack(stack_name, template_body):
    cf_client = boto3.client('cloudformation')
    response = cf_client.create_stack(
        StackName=stack_name,
        TemplateBody=template_body,
        Capabilities=['CAPABILITY_NAMED_IAM'],
        OnFailure='ROLLBACK'
    )
    return response
```

**Example input / output:**

```text
Input:
- RTO = 5 minutes, RPO = 30 seconds

Output:
- Backup strategy: Incremental backups every 5 minutes during business hours, full backup once per day at night.
- Failover strategy: Automated failover in another AWS region using CloudFormation templates and Route 53 for DNS health checks.

Test Results:
- RTO achieved within expected time frame.
- RPO met by regularly updating the secondary region with real-time data replication.
```

**Data:**

```json
{
  "BackupPlan": {
    "Type": "Object",
    "Properties": {
      "Frequency": "5 minutes during business hours, once per day at night",
      "RetentionPeriod": "30 days"
    }
  },
  "FailoverPlan": {
    "Type": "AWS::Route53::HealthCheck",
    "Properties": {
      "DNSResolverEndpointId": "string",
      "FQDN": "string",
      "FullyQualifiedDomainName": "string",
      "HealthThreshold": 2,
      "InsufficientDataHealthStatus": "string",
      "MeasureLatency": true,
      "Region": "us-west-1",
      "ResourcePath": "/path/to/resource",
      "SearchString": "string",
      "Type": "HTTP"
    }
  }
}
```

**Flow:**

1. Customer Request
2. Check RTO/RPO Gaps
3. Define Current State
4. Define Desired RTO & RPO
5. Implement Backup Strategy
6. Design Failover Mechanism
7. Regularly Test Backups
8. Maintain and Monitor

