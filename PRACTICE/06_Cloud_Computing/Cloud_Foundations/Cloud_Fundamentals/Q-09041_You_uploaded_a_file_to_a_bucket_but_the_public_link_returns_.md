---
type: "practice"
difficulty: "Intermediate"
---

# You uploaded a file to a bucket but the public link returns access denied. What are the likely causes?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Instance Roles and Access](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Instance_Roles_and_Access.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When you encounter an issue where a publicly shared link to a file in a cloud storage bucket returns "access denied," it usually points to one of several common issues. In this context, we need to ensure that our permissions and settings are correctly configured. Let's break down the potential causes and how to resolve them step by step.

**Comparison:**

```
Issue	Description
Incorrect Permissions	The object or bucket has misconfigured permissions.
Missing Public Link	The object might not have a publicly accessible URL.
IAM Role Issues	The IAM role associated with the S3 bucket might lack necessary permissions.
CORS Configuration	Cross-Origin Resource Sharing (CORS) settings are incorrectly configured.
Bucket Policy	The bucket policy might be overly restrictive or lacks the correct actions and principals.
```

**Code:**

```python
# Example code to check S3 bucket policies and IAM roles (pseudocode)
def check_bucket_policy(bucket_name):
    # This function would interact with AWS SDK or API to fetch policy details
    pass

def verify_iam_role(role_name):
    # This function checks the role's permissions in AWS IAM
    pass
```

**Example input / output:**

```text
Given:
- Bucket name: `example-bucket`
- Public link: `https://s3.amazonaws.com/example-bucket/file.pdf`

Expected output:
- Access to the file via the public link should be permitted.

Actual result:
- "Access denied" when trying to access the file through the public link.
```

**Data:**

```json
{
  "bucketName": "example-bucket",
  "publicLink": "https://s3.amazonaws.com/example-bucket/file.pdf"
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Check if the object has a publiclink
6. Yes?
7. Access via URL works
8. Generate or update public link
9. Check Bucket Policy forrestrictive permissions
10. Permissions OK?
11. Verify IAM Role Permissions
12. Update Bucket Policy to allowaccess
13. IAM Role Issues: Checkpermissions and policies
14. Correct IAM Role Policies
15. CORS Configuration: Check ifCORS is properly set up
16. Fix CORS configuration

