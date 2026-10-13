---
type: "practice"
difficulty: "Deep"
---

# An interviewer asks why you would not just give every service the same broad role 'to avoid breaking things during an incident'. How do you argue least-privilege without making on-call miserable?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Identity and Access Foundations](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Identity_and_Access_Foundations/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this question, we need to discuss the principles of least-privilege security and how it can be applied in a way that minimizes impact on on-call responsibilities. The key idea here is to balance security best practices with operational efficiency and team well-being.

The interviewer is probing for an understanding of how to implement granular permissions and roles, ensuring that services have only the necessary privileges while still maintaining operational resilience. We must consider both the technical and human aspects of managing cloud infrastructure securely.

Firstly, let's break down the concept: "least-privilege" means that each service or user should have the minimum level of access required to perform its tasks. This principle helps in minimizing the attack surface but can complicate on-call responsibilities if not properly managed.

To implement least-privilege effectively without making on-call miserable, we need a multi-layered approach:

Role-Based Access Control (RBAC): Define specific roles and permissions for each service based on its function.
Least Privilege Enforcement: Ensure that every role has the minimum necessary permissions to perform its tasks.
Access Reviews: Regularly review and update access controls to ensure they are still appropriate.

Let's dive into a step-by-step solution:

Step 1: Define Roles

Identify different roles within your infrastructure, such as DevOps Engineer, Site Reliability Engineer (SRE), On-Call Engineer, etc. Each role should have its own set of permissions tailored to their responsibilities.

Step 2: Implement RBAC

Use cloud provider features like AWS IAM, GCP IAM, or Azure AD to define roles and assign them permissions. For example:

YAML
# Example YAML for an SRE role in AWS IAM
---
RoleName: SRE-Role
Policies:
  - PolicyArn: arn:aws:iam::123456789012:policy/SRE-SecurityPolicy
  - PolicyArn: arn:aws:iam::123456789012:policy/DevOps-AccessPolicy
Step 3: Apply Granular Permissions

Grant permissions based on the specific needs of each role. For instance, an On-Call Engineer should have minimal access to prod environments but full control over staging and dev.

YAML
# Example YAML for On-Call Engineer's role in AWS IAM
---
RoleName: On-Call-Engineer
Policies:
  - PolicyArn: arn:aws:iam::123456789012:policy/On-Call-SecurityPolicy
Step 4: Monitor and Audit Access

Implement monitoring tools to track who is accessing what resources. Use audit logs to detect any unauthorized access or suspicious activity.

BASH
# Example command to enable AWS CloudTrail for auditing
aws cloudtrail create-trail Name=MyTrail S3BucketName=my-bucket
Step 5: On-Call Management Strategy

Ensure that on-call engineers have the necessary tools and support during incidents. Provide them with a well-documented incident response plan, access to monitoring tools, and regular training.

MARKDOWN
# Example Incident Response Plan
- **Alerting:** Use alerting systems like Prometheus + Grafana or ELK Stack.
- **Communication Tools:** Slack channels for quick communication.
- **Documentation:** Access to detailed documentation of services and infrastructure.
Step 6: Regular Reviews and Updates

Regularly review access policies and update them as needed. This ensures that the least-privilege model remains effective over time.

#### Closing Summary

By implementing a well-defined RBAC structure, applying granular permissions, monitoring access, providing necessary tools and support for on-call engineers, and regularly reviewing policies, we can achieve both security and operational efficiency. This approach minimizes the risk of breaking things during an incident while ensuring that on-call responsibilities are manageable.

#### Common Mistakes/Follow-Ups
Over-Privileged Roles: Failing to define roles granularly can lead to overprivileged access.
Lack of Monitoring: Not implementing robust monitoring can make it difficult to detect and respond to security breaches.
Insufficient Training: On-call engineers may struggle if they lack the necessary training or tools.

By addressing these points, we can argue effectively for a balanced approach that adheres to least-privilege principles while keeping on-call operations smooth and effective.

**Comparison:**

| Role            | Permissions                |
|-----------------|----------------------------|
| SRE             | Full access to prod        |
| On-Call Engineer| Minimal access to prod    |
| DevOps          | Access to staging, dev     |
| Monitoring Tools | Description                  |
|------------------|------------------------------|
| Prometheus       | Metrics collection and alerting   |
| Grafana          | Visualization of metrics         |
| Slack            | Communication for incidents     |

**Code:**

```yaml
# Example IAM role definition in YAML format
---
RoleName: SRE-Role
Policies:
  - PolicyArn: arn:aws:iam::123456789012:policy/SRE-SecurityPolicy
  - PolicyArn: arn:aws:iam::123456789012:policy/DevOps-AccessPolicy

# Example IAM role definition in YAML format for On-Call Engineer
---
RoleName: On-Call-Engineer
Policies:
  - PolicyArn: arn:aws:iam::123456789012:policy/On-Call-SecurityPolicy
```

**Example input / output:**

```text
# Example of a dry-run input and expected output for IAM role creation
Input:
aws iam create-role \
    --role-name On-Call-Engineer \
    --assume-role-policy-document file://trust-policy.json \
    --path /on-call/

Output:
{
  "Role": {
    "AssumeRolePolicyDocument": {...},
    "RoleId": "AROA1234567890",
    ...
  }
}

# Example of a CloudTrail enablement input and output
Input:
aws cloudtrail create-trail \
    --name MyTrail \
    --s3-bucket-name my-bucket

Output:
{
  "Name": "MyTrail",
  "S3BucketName": "my-bucket"
}
```

**Data:**

```yaml
# Example of a simple IAM policy definition in YAML
---
Version: "2012-10-17"
Statement:
  - Sid: AllowReadS3Bucket
    Effect: Allow
    Action: s3:GetObject
    Resource: arn:aws:s3:::my-bucket/*

# Example of a CloudTrail configuration in YAML
---
CloudWatchLogsLogGroupArn: arn:aws:logs:us-west-2:123456789012:log-group:/aws/cloudtrail/my-trail
Enabled: true

# Example of an incident response plan excerpt
# Alerting and communication tools setup in a fictional system
---
Alerts:
  - When: Network error detected
    Action: Trigger Slack alert to #ops-channel
CommunicationTools:
  - ToolName: Slack
    Channel: #incident-response
Documentation:
  - ServiceSpec: S3-Bucket-Management.md
  - InfrastructureDoc: cloud-infrastructure-diagram.pdf
```

**Flow:**

1. Yes
2. No
3. Define Roles
4. Are roles defined?
5. Implement RBAC
6. Refine Role Definitions
7. Apply Granular Permissions
8. Monitor and Audit Access
9. Ensure On-Call Support Tools
10. Regular Reviews and Updates

