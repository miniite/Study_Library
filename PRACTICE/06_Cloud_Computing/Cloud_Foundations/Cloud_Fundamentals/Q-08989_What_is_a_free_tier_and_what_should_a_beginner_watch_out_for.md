---
type: "practice"
difficulty: "Basic"
---

# What is a free tier and what should a beginner watch out for so they do not get a surprise bill?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Cloud Cost Attribution](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Cloud_Cost_Attribution.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In cloud computing, the "free tier" refers to the set of services or resources that providers offer at no cost to new users. For beginners, understanding what constitutes the free tier and how to manage usage is crucial to avoid unexpected charges. This overview will cover the basics of free tiers, common pitfalls, and best practices.

To start with, let's clarify what a free tier actually means:

Free Tier: Cloud providers offer a limited set of services or resources at no cost for a certain period (often 30 days) after account creation.
Usage Limits: These limits can vary widely by provider and service. Exceeding these limits often results in additional charges.

Now, let's dive into the details of what beginners should watch out for:

Resource Limits: Many free tiers have strict resource usage limits. For example, storage space might be limited to 5 GB or a certain amount of CPU/GPU hours.
Data Transfer: Free data transfer is often provided within a region but may incur charges when transferring data across regions or internationally.
Termination Fees and Cleanup: Some providers charge for terminating services or require manual cleanup, which can lead to unnoticed costs if not properly managed.
Hidden Costs: Not all features are included in the free tier. Additional components like monitoring tools might be available only with paid plans.

A beginner should take these steps to avoid surprises:

Review Documentation: Thoroughly read the provider's documentation for your specific services and understand what is included in the free tier.
Monitor Usage: Regularly check usage statistics through cloud console or APIs.
Automate Cleanup: Use scripts to automatically terminate unused resources when a project ends.
Set Up Alerts: Configure alerts to notify you of approaching resource limits.

In conclusion, while the free tier can be a great starting point for beginners, it's essential to have a clear understanding of its limitations and potential costs. Careful management and proactive monitoring are key to avoiding unexpected bills.

**Comparison:**

```
Cloud Provider	Free Tier Storage Limit (GB)	Data Transfer (Local/Regional)
AWS	5	Free
GCP	10	$0.04/GB
Azure	20	Included within subscription
```

**Code:**

```text
# Example of setting up AWS CloudWatch alerts (Pseudo-code)
import boto3

def set_alerts():
    client = boto3.client('cloudwatch')

    # Set CPU utilization alarm
    response = client.put_metric_alarm(
        AlarmName='HighCPUUtilization',
        MetricName='CPUUtilization',
        Namespace='AWS/EC2',
        Threshold=70.0,
        Period=60,
        EvaluationPeriods=1,
        Statistic='Average',
        ComparisonOperator='GreaterThanThreshold',
        ActionsEnabled=True
    )

    # Set disk space alarm
    response = client.put_metric_alarm(
        AlarmName='LowDiskSpace',
        MetricName='FreeDiskSpace',
        Namespace='AWS/ElasticBlockStore',
        Threshold=500,
        Period=60,
        EvaluationPeriods=1,
        Statistic='Minimum',
        ComparisonOperator='LessThanThreshold',
        ActionsEnabled=True
    )
```

**Example input / output:**

```text
Input:
- AWS Free Tier Resources: S3 Storage Limit = 5 GB, EBS Volume Size = 8 GB, RDS Instance Hours = 750 hours

Output:
- S3 storage usage alert triggered when approaching 4.9 GB
- EBS volume size limit exceeded warning sent to the user after reaching 7.2 GB
- RDS instance hour threshold near 749 hours, requiring early termination or adjustment of resources
```

**Data:**

```json
{
  "freeTierResources": {
    "S3StorageLimitGB": 5,
    "EBSVolumeSizeGB": 8,
    "RDSInstanceHours": 750,
    "VPCs": 1,
    "SecurityGroups": 20,
    "LambdaInvocations": 1M
  },
  "dataTransferLimits": {
    "LocalRegionDataTransfer": true,
    "CrossRegionDataTransferRate": "$0.01/GB",
    "InternationalDataTransferRate": "$0.09/GB"
  }
}
```

**Flow:**

1. Check free tier resources
2. Exceeded limits?
3. Set up alerts and monitorusage
4. Terminate unused servicesautomatically
5. Configure alerts for resourcethresholds

