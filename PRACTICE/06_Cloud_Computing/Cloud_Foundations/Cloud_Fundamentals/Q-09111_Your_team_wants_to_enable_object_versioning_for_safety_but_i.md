---
type: "practice"
difficulty: "Intermediate"
---

# Your team wants to enable object versioning for safety but is worried about storage cost. How do you reconcile the two with lifecycle rules, and what recovery scenario justifies the extra spend?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Object Storage Classes](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Object_Storage_Classes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address your team's requirement of enabling object versioning while managing costs, we need to strike a balance between storage efficiency and data safety through lifecycle policies and cost-effective strategies. This involves understanding the nuances of Amazon S3 Lifecycle policies and considering alternative storage classes that can help manage costs. Let’s break down the solution step-by-step.

#### Triaging the Question

The core issue here is maintaining object versioning for safety without incurring excessive storage costs. The interviewer is probing your knowledge of S3 lifecycle management, cost optimization techniques, and recovery scenarios. We need to make assumptions about the expected usage patterns, retention periods, and the nature of data that can be stored in different storage classes.

#### Solution
from First Principles
Enable Object Versioning: This ensures that every version of an object is saved, providing a safety net for accidental deletions or overwrites.
Use S3 Lifecycle Policies: Define lifecycle rules to automatically transition older versions of objects to cheaper storage tiers after a certain period.
Cost-Effective Storage Classes: Utilize Infrequent Access (IA) and Glacier classes for long-term archiving, which are significantly cheaper than Standard storage but still provide durable data protection.
Step-by-Step Solution

Enable Object Versioning:

Enable versioning on the S3 bucket to keep all versions of each object.
This ensures that every change to an object is saved and can be recovered if needed.

Define Lifecycle Policies:

Create lifecycle rules to transition old versions to cheaper storage classes after a specific number of days (e.g., 30, 60, or 90 days).

#### Example
policy:
JSON
{
  "Rules": [
    {
      "ID": "ArchiveOldObjects",
      "Status": "Enabled",
      "Prefix": "",
      "Transitions": [
        {
          "StorageClass": "STANDARD_IA",
          "Days": 30
        },
        {
          "StorageClass": "GLACIER",
          "Date": "2025-12-31T00:00:00Z"
        }
      ],
      "NoncurrentVersionTransitions": [
        {
          "StorageClass": "STANDARD_IA",
          "NoncurrentDays": 90
        },
        {
          "StorageClass": "GLACIER",
          "NoncurrentDate": "2030-12-31T00:00:00Z"
        }
      ]
    }
  ]
}

Cost Optimization:

Use the cheaper storage classes (STANDARD_IA, GLACIER) for long-term archival needs.
Keep recent versions in STANDARD or one of the higher-tier storage classes to ensure faster access and availability.

Recovery Scenario:

A justified scenario where extra spend is warranted might be a critical data set that requires frequent access and cannot afford any downtime. For instance, if your team deals with regulatory compliance requirements that necessitate quick recovery of old versions.
In such cases, you can keep the most recent 30-60 days in STANDARD storage while transitioning older data to IA or Glacier.

#### Closing Summary

In summary, enabling object versioning and using S3 Lifecycle policies is a balanced approach to ensuring safety and managing costs. By leveraging different storage classes based on access patterns and retention needs, you can optimize costs without compromising on the ability to recover objects when needed.

#### Common Mistakes/Follow-Ups
Misunderstanding Retention Periods: Failing to set appropriate transition periods for lifecycle policies might lead to unnecessary data being stored in expensive classes.
Ignoring Access Patterns: Not considering how frequently data is accessed can result in higher costs due to keeping all versions in more expensive storage classes unnecessarily.
Overlooking Compliance Requirements: Neglecting specific compliance needs might mean that critical data sets are not properly protected, leading to recovery issues.

By carefully planning and implementing these strategies, you can ensure both safety and cost-effectiveness for your team’s use of S3 object versioning.

**Code:**

```json
{
  "Rules": [
    {
      "ID": "ArchiveOldObjects",
      "Status": "Enabled",
      "Prefix": "",
      "Transitions": [
        {
          "StorageClass": "STANDARD_IA",
          "Days": 30
        },
        {
          "StorageClass": "GLACIER",
          "Date": "2025-12-31T00:00:00Z"
        }
      ],
      "NoncurrentVersionTransitions": [
        {
          "StorageClass": "STANDARD_IA",
          "NoncurrentDays": 90
        },
        {
          "StorageClass": "GLACIER",
          "NoncurrentDate": "2030-12-31T00:00:00Z"
        }
      ]
    }
  ]
}
```

**Example input / output:**

```text
Input:
{
  "Rules": [
    {
      "ID": "ArchiveOldObjects",
      "Status": "Enabled",
      "Prefix": "",
      "Transitions": [
        {
          "StorageClass": "STANDARD_IA",
          "Days": 30
        },
        {
          "StorageClass": "GLACIER",
          "Date": "2025-12-31T00:00:00Z"
        }
      ],
      "NoncurrentVersionTransitions": [
        {
          "StorageClass": "STANDARD_IA",
          "NoncurrentDays": 90
        },
        {
          "StorageClass": "GLACIER",
          "NoncurrentDate": "2030-12-31T00:00:00Z"
        }
      ]
    }
  ]
}

Output:
Lifecycle policy created successfully.
```

