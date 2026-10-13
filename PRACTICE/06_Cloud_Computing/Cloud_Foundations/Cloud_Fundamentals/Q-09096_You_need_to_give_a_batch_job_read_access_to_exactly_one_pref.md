---
type: "practice"
difficulty: "Intermediate"
---

# You need to give a batch job read access to exactly one prefix of one bucket. Walk me through writing that IAM policy and how you would test it does not grant more than intended.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Instance Roles and Access](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Instance_Roles_and_Access.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

To ensure a batch job has read access to exactly one prefix within an S3 bucket, we'll write an IAM policy that grants minimal necessary permissions. We’ll then test this policy to confirm it restricts access as intended.

#### Triaging the Question

The question asks for an IAM (Identity and Access Management) policy that gives read-only access to a specific prefix in an S3 bucket. The key points are:

Bucket: Exactly one.
Prefix: Exactly one.
Read Access Only.
Ensure No More Than Intended Access.

#### Assumptions
We use AWS IAM and S3 services.
The batch job will run from an EC2 instance or a serverless function, e.g., AWS Lambda.
The policy needs to be tested in a controlled environment without impacting real data.

#### Solution
Writing the IAM Policy
Step 1: Define the Prefix

Let’s assume our S3 bucket is named my-cool-bucket, and we want to grant read access only to objects under the prefix data/2023/q4.

PLAINTEXT
Prefix: /data/2023/q4/
Step 2: Create a Policy Document

The policy should have the following structure:

Principal: The identity that will be granted this permission.
Effect: Allow for read operations (s3:GetObject).
Action: s3:GetObject.
Resource: A specific S3 prefix.

Here’s a sample IAM policy document in JSON format:

JSON
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowReadAccessToData2023Q4",
      "Effect": "Allow",
      "Action": ["s3:GetObject"],
      "Resource": "arn:aws:s3:::my-cool-bucket/data/2023/q4/*"
    }
  ]
}
Step 3: Testing the Policy

To ensure this policy restricts access as intended, we can follow these steps:

Create a Test S3 Bucket and Objects:

Create my-cool-bucket with objects at different paths.
Ensure there are no files in other prefixes under data/2023/q4.

Attach the Policy to an IAM Role or User:

Attach this policy to a specific user or role that represents your batch job.

Run the Batch Job and Test Access:

Use the AWS CLI, SDKs, or any tool from within the EC2 instance or serverless function to list objects under /data/2023/q4.
Ensure only files under this path are accessible.
Step 4: Validate No More Than Intended Access
Check All Prefixes: Ensure no other prefixes like data/2023/q1 or logs/2023/q4 can be accessed.
Use S3 Management Tools: Use AWS Console, CLI commands, or SDKs to verify the policy works as intended.
Full Policy Document

Here’s a complete example of the IAM policy:

JSON
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowReadAccessToData2023Q4",
      "Effect": "Allow",
      "Action": ["s3:GetObject"],
      "Resource": "arn:aws:s3:::my-cool-bucket/data/2023/q4/*"
    }
  ]
}

#### Closing Summary

By following these steps, we ensure the IAM policy grants read access only to a specific prefix in an S3 bucket. Testing is critical to verify no unintended access.

#### Common Mistakes/Follow-Ups
Incorrect ARN: Always double-check the ARN format for the resource.
Overly Broad Policies: Avoid granting broader permissions than necessary.
Inconsistent Testing Environment: Ensure testing doesn’t impact production data or services.
Policy Inheritance: Consider if any other policies might override this one.

**Code:**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowReadAccessToData2023Q4",
      "Effect": "Allow",
      "Action": ["s3:GetObject"],
      "Resource": "arn:aws:s3:::my-cool-bucket/data/2023/q4/*"
    }
  ]
}
```

