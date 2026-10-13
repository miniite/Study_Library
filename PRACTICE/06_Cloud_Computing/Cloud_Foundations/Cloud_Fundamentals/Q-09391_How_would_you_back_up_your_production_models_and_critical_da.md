---
type: "practice"
difficulty: "Intermediate"
---

# How would you back up your production models and critical datasets so an accidental deletion in the cloud is recoverable?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Object Storage Classes](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Object_Storage_Classes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that production models and critical datasets are recoverable from accidental deletions in the cloud, we need a robust backup strategy. This involves multiple steps such as regular backups, versioning, lifecycle management, and redundancy across different regions or availability zones.

#### Triage

The interviewer is probing your understanding of cloud storage best practices to prevent data loss due to human error. Key assumptions include the use of a managed service for backups, like Amazon S3 with Versioning enabled, and leveraging cloud-native tools for automated backup processes.

#### First-Principles Solution

Enable Versioning: Start by enabling versioning on your storage bucket in AWS S3 or Azure Blob Storage. This ensures that every upload or update to a file creates a new version, which can be restored if needed.

Automated Backups: Use cloud-native tools like AWS Backup or Google Cloud DLP (Data Loss Prevention) for automated backups. These tools allow you to schedule regular snapshots and backups of your datasets and models.

Lifecycle Management: Implement lifecycle policies to manage the retention and deletion of backups based on their age. For example, keep daily backups for a month, weekly backups for a year, and yearly backups indefinitely.

Redundancy Across Regions: To ensure high availability in case of regional outages or disasters, store your backup data across multiple regions using cross-region replication.

Testing and Validation: Regularly test the restore process to ensure that you can recover your models and datasets when needed. This helps identify any issues with the backup strategy before a critical situation arises.

#### Closing Summary

Implementing a comprehensive backup strategy involves enabling versioning, automating backups, managing the lifecycle of backups, ensuring redundancy across regions, and regularly testing restores. These steps help mitigate risks associated with accidental deletions in the cloud.

#### Common Mistakes/Follow-Ups
Ignoring Versioning: Failing to enable versioning can result in data loss if a mistake is made during an update.
Lack of Testing: Not testing your backup and restore processes can lead to failure when you need to recover data.
Inadequate Retention Policies: Insufficient retention policies may cause older backups to be deleted before they are needed.

**Comparison:**

```
Backup Strategy	Description
Enable Versioning	Ensures data retention with every update.
Automated Backups	Uses cloud-native tools to schedule regular backups.
Lifecycle Management	Manages the retention and deletion of backups based on age.
Redundancy Across Regions	Stores backup data across multiple regions for high availability.
```

**Code:**

```python
# Example Python code snippet for setting up versioning in AWS S3
import boto3

s3 = boto3.client('s3')
response = s3.put_bucket_versioning(
    Bucket='your-bucket-name',
    VersioningConfiguration={
        'Status': 'Enabled'
    }
)
```

**Example input / output:**

```text
Input:
- Enable versioning in S3 bucket named 'models-backup'.

Output:
- Versioning is enabled.
- Lifecycle rule created to keep daily backups for 30 days.

Dry-run request for a model file upload and delete operation followed by restore from backup.
```

**Data:**

```json
{
  "Bucket": "models-backup",
  "Versioning": {
    "Status": "Enabled"
  },
  "LifecycleRules": [
    {
      "ID": "DailyBackupRule",
      "Prefix": "",
      "Status": "Enabled",
      "NoncurrentVersionExpiration": {"Days": 30},
      "Transition": []
    }
  ]
}
```

**Flow:**

1. Upload Model
2. Versioning Enabled
3. New Version Created
4. Lifecycle Rule Applies
5. Keep Backup for 30 Days
6. Delete Original Model
7. Backup Restored from LatestVersion

