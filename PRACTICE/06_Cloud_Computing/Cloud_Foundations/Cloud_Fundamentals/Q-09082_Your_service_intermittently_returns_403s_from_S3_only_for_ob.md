---
type: "practice"
difficulty: "Deep"
---

# Your service intermittently returns 403s from S3 only for objects uploaded in the last hour while older objects work fine. How would you determine whether it is a bucket policy, KMS key grant, or eventual-consistency-on-a-recently-changed-ACL issue?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Instance Roles and Access](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Instance_Roles_and_Access.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To diagnose why your service intermittently returns 403s from S3 only for objects uploaded in the last hour, while older objects work fine, we need to systematically investigate potential issues like bucket policy, KMS key grants, or eventual consistency on recently changed ACLs. We’ll approach this step-by-step using first principles.

First, let's clarify what each of these could potentially cause:

Bucket Policy: Incorrect permissions in the bucket policy might be denying access.
KMS Key Grants: Permissions related to encryption keys (if used) might not be correctly set up.
Eventual Consistency on ACLs: Changes to object ACLs may not have propagated fully, leading to transient issues.

**Comparison:**

```
Key	Value
Bucket Name	my-bucket
Key ID	alias/my-key
```

**Code:**

```python
# Example function to check S3 bucket policy and KMS key grants using boto3 in Python
import boto3

def check_s3_policy(bucket_name):
    s3_client = boto3.client('s3')
    policy = s3_client.get_bucket_policy(Bucket=bucket_name)
    print(f"Bucket Policy: {policy}")

def check_kms_grants(key_id):
    kms_client = boto3.client('kms')
    grants = kms_client.list_grants(KeyId=key_id)
    print(f"KMS Grants for key {key_id}: {grants}")

# Example usage
bucket_name = 'my-bucket'
key_id = 'alias/my-key'
check_s3_policy(bucket_name)
check_kms_grants(key_id)
```

**Example input / output:**

```
Input: bucket_name = 'my-bucket', key_id = 'alias/my-key'
Output:

Bucket Policy: { ... (example policy JSON) } KMS Grants for key my-key: [ {...} ]
```

**Data:**

```json
{
  "bucket_name": "my-bucket",
  "key_id": "alias/my-key"
}
```

**Flow:**

1. Yes
2. Yes
3. No
4. No
5. Check Bucket Policy
6. Is policy restrictive?
7. Inspect ACLs and grants
8. Are recent changes in ACLsaffecting permissions?
9. Investigate consistency of ACLupdates
10. Review KMS key grants for thisobject
11. Permissions are correct, lookelsewhere

