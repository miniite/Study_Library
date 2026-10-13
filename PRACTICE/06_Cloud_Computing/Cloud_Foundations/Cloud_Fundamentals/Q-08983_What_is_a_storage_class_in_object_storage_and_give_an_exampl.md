---
type: "practice"
difficulty: "Basic"
---

# What is a storage class in object storage and give an example of when you would use a cheaper class.

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Object Storage Classes](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Object_Storage_Classes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Storage classes in cloud object storage are predefined policies that define the performance, durability, and cost characteristics of data stored. They help organizations balance their needs with costs by allowing them to store data at different levels depending on access frequency and importance. For example, a cheaper class might be used for long-term archival or backup purposes.

To triage this question:

The interviewer is probing your understanding of storage classes and their usage in cloud object storage.
You need to explain the concept of storage classes and provide an example of a cost-effective option.

Now, let's reason through this step by step from first principles:

Definition: A storage class in cloud object storage defines how data will be stored based on specific requirements such as performance, durability, and cost.
Types of Storage Classes:
Performance Classes: These include options like Standard (faster access) and Infrequent Access (slower but more frequent access).
Cost-Efficient Classes: These are designed for infrequent access or long-term archival where performance is less critical, such as Cold Archive.

#### Example
of a Cheaper Class:
AWS S3 Standard-Infrequent Access (S3 Standard IA): This class provides the same durability and availability guarantees as S3 Standard but with lower storage costs. It's suitable for data that needs to be accessed infrequently, such as backups or older archives.

Let's break it down further:

Durability: Both S3 Standard and S3 Standard-IA provide 99.999999999% durability.
Performance: S3 Standard offers faster access times (expected to be faster than S3 Standard-IA).
Cost: S3 Standard-IA is significantly cheaper, making it ideal for long-term storage where the data might not need frequent access.

Here’s a simple example of how you might use this class:

Imagine an organization has backups of their database that are only accessed once every few months. Instead of using the more expensive S3 Standard, they can use S3 Standard-IA to store these backups and save on storage costs while still ensuring data is available when needed.

In conclusion, understanding storage classes helps in optimizing cloud storage costs without compromising on data availability or security. The key takeaway is that cheaper classes like S3 Standard-IA are designed for less frequent access scenarios, making them cost-effective options for long-term storage needs.

**Comparison:**

```
Storage Class	Durability (%)	Performance	Cost
S3 Standard	99.999999999%	Fast	High
S3 Standard-IA	99.999999999%	Slow	Low
```

**Code:**

```
MARKDOWN
Copy
# Example of using a cheaper class (AWS CLI)
aws s3api put-object --bucket my-backup-bucket --key backup-2023-10-01.sql.gz --storage-class Standard-Infrequent-Access
```

**Example input / output:**

```

Request:

SH
Copy
aws s3api put-object --bucket my-backup-bucket --key backup-2023-10-01.sql.gz --storage-class Standard-Infrequent-Access

Response:

JSON
Copy
{
  "ETag": "\"d41d8cd98f00b204e9800998ecf8427e\"",
  "VersionId": "<unique version identifier>"
}
```

**Data:**

```json
{
  "Bucket": "my-backup-bucket",
  "Key": "backup-2023-10-01.sql.gz",
  "StorageClass": "Standard-Infrequent-Access"
}
```

**Flow:**

1. No
2. Yes
3. User
4. Create Backup
5. Is Data Access Frequent?
6. S3 Standard-IA
7. S3 Standard

