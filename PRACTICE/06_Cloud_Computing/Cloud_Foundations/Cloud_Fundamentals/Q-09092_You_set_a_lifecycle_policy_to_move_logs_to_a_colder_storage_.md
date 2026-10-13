---
type: "practice"
difficulty: "Intermediate"
---

# You set a lifecycle policy to move logs to a colder storage tier after 30 days and now your monthly retrieval spikes cost more than the storage you saved. How do you diagnose and re-tune the policy?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Cold Starts and Regional Differences](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Cold_Starts_and_Regional_Differences.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this scenario, we need to diagnose why our logs are being retrieved more frequently than expected after 30 days, leading to higher retrieval costs despite lower storage costs. We'll start by understanding the lifecycle policy and then proceed step-by-step to identify potential issues and re-tune the policy accordingly.

Firstly, let's summarize the core issue: a lifecycle policy intended to move logs to colder storage after 30 days is now causing higher retrieval costs due to frequent access to these logs. This suggests that the logs are being accessed more often than anticipated, or there might be misconfiguration in how the lifecycle rules are applied.

To diagnose this problem:

Review Lifecycle Policy Configuration: Check if the policy is correctly set up to move logs after 30 days and ensure it's being applied consistently across all relevant storage buckets.
Analyze Retrieval Logs: Examine AWS CloudTrail or equivalent service to see how often these logs are accessed before and after the 30-day mark.
Check User Access Patterns: Investigate if there are specific users or applications that frequently retrieve these older logs, which might not align with business needs.
Evaluate Policy Effectiveness: Ensure that the policy is not overly restrictive, preventing legitimate access to important data.

Now let's break this down from first principles:

Step-by-Step Solution

Review Lifecycle Policy Configuration

Log in to your AWS console or use the AWS CLI command aws s3api get-lifecycle-configuration --bucket <bucket-name> to retrieve the lifecycle policy details.
Ensure that the rules are correctly set to move objects to colder storage tiers (e.g., S3 Standard-Infrequent Access, Glacier) after 30 days. The rule should look something like:
JSON
{
  "Rules": [
    {
      "ID": "MoveLogsToColdTier",
      "Prefix": "logs/",
      "Status": "Enabled",
      "Transitions": [
        {
          "Days": 30,
          "StorageClass": "GLACIER"
        }
      ],
      "NoncurrentVersionTransitions": []
    }
  ]
}

Analyze Retrieval Logs

Use AWS CloudTrail or S3 Analytics to monitor access patterns.
Run a query like:
SQL
SELECT * FROM s3_access_logs WHERE bucket_name = 'your-bucket-name' AND prefix LIKE 'logs/%';
Look for unusual access patterns, such as frequent retrieval of older logs.

Check User Access Patterns

Identify which users or applications are accessing these logs frequently.
For example, if a specific application is consistently retrieving old logs, it might be due to a bug in the application logic.

Evaluate Policy Effectiveness

Ensure that the policy isn't overly restrictive and doesn’t prevent legitimate access.

#### Cons
ider adjusting the lifecycle policy based on actual usage patterns, possibly extending the transition period or adding exceptions for critical data.

#### Closing Summary

By following these steps, we can diagnose why our logs are being retrieved more frequently than expected and re-tune the lifecycle policy to better balance storage costs with access needs. Common mistakes include misconfiguring the lifecycle rules, overlooking user access patterns, and not regularly reviewing the effectiveness of the policy.

#### Common Mistakes/Follow-Ups
Misconfigured Lifecycle Policy: Ensure that the rules are correctly set up and applied.
Unusual Access Patterns: Investigate any unexpected spikes in log retrievals to understand the root cause.
Overly Restrictive Policies: Balance cost savings with legitimate access needs, possibly by adjusting transition periods or adding exceptions.

**Comparison:**

```
Rule ID	Prefix	Status	Days	StorageClass
MoveLogsToColdTier	logs/	Enabled	30	GLACIER
```

**Code:**

```json
{
  "Rules": [
    {
      "ID": "MoveLogsToColdTier",
      "Prefix": "logs/",
      "Status": "Enabled",
      "Transitions": [
        {
          "Days": 30,
          "StorageClass": "GLACIER"
        }
      ],
      "NoncurrentVersionTransitions": []
    }
  ]
}
```

**Example input / output:**

```text
- **Input**: `aws s3api get-lifecycle-configuration --bucket logs-bucket`
- **Output**: Lifecycle policy details
```

**Data:**

```sql
SELECT * FROM s3_access_logs WHERE bucket_name = 'your-bucket-name' AND prefix LIKE 'logs/%';
```

**Flow:**

1. Review Lifecycle Policy
2. Analyze Retrieval Logs
3. Check User Access Patterns
4. Evaluate Policy Effectiveness
5. Tune Policy Based on Findings

