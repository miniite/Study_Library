---
type: "practice"
difficulty: "Expert"
---

# How would you architect a cloud landing zone from scratch for a regulated fintech expecting to enter five countries, so that data residency, guardrails, and account structure are correct on day one rather than retrofitted?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Data Lifecycle and Retention](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Stateless_by_Default/Data_Lifecycle_and_Retention.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To architect a cloud landing zone for a regulated fintech company entering five countries, we need to ensure compliance with data residency regulations, implement robust guardrails, and establish a proper account structure. This involves designing a scalable, secure, and regulatory-compliant environment that is ready from the outset.

The key steps involve:

Understanding Regulatory Requirements: Identify specific data residency laws for each country.
Designing an Account Structure: Create separate accounts or resources to segregate different environments (e.g., development, staging, production).
Implementing Guardrails and Compliance Controls: Ensure that access controls, encryption, auditing, and other security measures are in place.
Setting Up Data Residency: Configure storage services to ensure data is stored within the appropriate geographic region.

Let’s break this down from first principles.

#### Triage
What We’re Probing

The interviewer is probing your understanding of:

Regulatory compliance requirements (specifically data residency laws).
Best practices in cloud architecture for multi-region deployments.
Experience with implementing guardrails and security controls.
Ability to design a scalable, secure, and compliant solution from scratch.

#### Assumptions
Regulatory Environment: Countries have varying levels of regulatory oversight regarding data storage.
Cloud Services: Use of major cloud providers like AWS, Azure, or Google Cloud Platform (GCP).
Existing Tools: Availability of tools for network segmentation, access controls, and monitoring.
Step-by-Step Solution
1. Understanding Regulatory Requirements

First, we need to identify the data residency requirements for each country. For example:

Country A requires all personal data to be stored within its borders.
Country B allows cross-border data flows but mandates strong encryption and access controls.
Countries C, D, and E have specific compliance frameworks that must be followed.
2. Designing an Account Structure

For a multi-country deployment, we can use AWS Organizations or Azure Active Directory (AAD) to manage multiple accounts while maintaining centralized governance:

Root Organization: Set up the root organization in a dedicated account for global policies.
Country-Specific Accounts: Create separate accounts for each country within the organization.
Cross-Account Access: Use cross-account roles and permissions to enable secure access between different environments.

#### Example
of an organization structure:

CODE
+---------------------+
| Global Policy       |
+---------------------+
          |
          v
+-------------+     +-------------+     +-------------+
| Account A   |     | Account B   |     | Account C   |
+-------------+     +-------------+     +-------------+
          |             |              |
          v             v              v
+-----------+  +-----------+  +-----------+
| Dev/Stage |  | Staging   |  | Production|
+-----------+  +-----------+  +-----------+
3. Implementing Guardrails and Compliance Controls

Implement a series of guardrails to enforce compliance:

Access Control: Use IAM policies, CIs, and RBAC.
Encryption: Enable encryption for storage (SSE) and in-transit data using TLS.
Auditing and Logging: Set up CloudTrail or Azure Monitor for auditing.
Security Groups and Network Segmentation: Ensure proper network segmentation to prevent unauthorized access.

#### Example
of IAM policy:

JSON
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyUnencryptedS3Access",
      "Effect": "Deny",
      "Action": ["s3:*"],
      "Resource": "*",
      "Condition": {"Bool": {"aws:SecureTransport": "false"}}
    }
  ]
}
4. Setting Up Data Residency

Configure data residency by setting up storage services within the required geographic regions:

S3 Buckets: Create S3 buckets in each country's designated region.
RDS Instances: Deploy RDS instances in local AWS regions or use a managed service like Google Cloud SQL.
Data Replication: Implement data replication strategies to ensure compliance with cross-border data flows.

#### Example
of an S3 bucket configuration:

YAML
Resources:
  DataBucketEU:
    Type: "AWS::S3::Bucket"
    Properties:
      BucketName: "data-residency-eu"
      PublicAccessBlockConfiguration:
        BlockPublicAcls: true
        BlockPublicPolicy: true
        IgnorePublicAcls: true
        RestrictPublicBuckets: true
  DataBucketUS:
    Type: "AWS::S3::Bucket"
    Properties:
      BucketName: "data-residency-us"
      PublicAccessBlockConfiguration:
        BlockPublicAcls: true
        BlockPublicPolicy: true
        IgnorePublicAcls: true
        RestrictPublicBuckets: true

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, the key is to design a robust, scalable architecture that not only meets regulatory requirements but also ensures data security and access control. The common pitfalls include:

Ignoring regional compliance laws.
Failing to implement proper encryption and network segmentation.
Not setting up comprehensive logging and auditing.

For follow-ups, you might be asked about specific tools or services used (e.g., AWS Config, Azure Policy), integration with on-premises infrastructure, or potential cost optimization strategies.

**Comparison:**

| Account | Region     | Compliance Framework |
|---------|------------|----------------------|
| A       | EU         | GDPR                  |
| B       | US         | CCPA                  |
| C       | Australia  | APRA                  |
| D       | Japan      | PDP                   |
| E       | Singapore  | PDPA                  |

**Code:**

```json
{
  "Resources": {
    "DataBucketEU": {
      "Type": "AWS::S3::Bucket",
      "Properties": {
        "BucketName": "data-residency-eu",
        "PublicAccessBlockConfiguration": {
          "BlockPublicAcls": true,
          "BlockPublicPolicy": true,
          "IgnorePublicAcls": true,
          "RestrictPublicBuckets": true
        }
      }
    },
    "DataBucketUS": {
      "Type": "AWS::S3::Bucket",
      "Properties": {
        "BucketName": "data-residency-us",
        "PublicAccessBlockConfiguration": {
          "BlockPublicAcls": true,
          "BlockPublicPolicy": true,
          "IgnorePublicAcls": true,
          "RestrictPublicBuckets": true
        }
      }
    }
  }
}
```

**Example input / output:**

```text
Input:
- Define regulatory requirements for data residency in each country.
- Set up an organization structure with separate accounts for each region.

Output:
- Root Organization account with global policies.
- Country-specific accounts (e.g., EU, US).
- IAM policies and roles configured to enforce access control.
- Encryption enabled on S3 buckets and RDS instances.
- CloudTrail or Azure Monitor logging setup.
```

**Data:**

```yaml
Resources:
  DataBucketEU:
    Type: "AWS::S3::Bucket"
    Properties:
      BucketName: "data-residency-eu"
      PublicAccessBlockConfiguration:
        BlockPublicAcls: true
        BlockPublicPolicy: true
        IgnorePublicAcls: true
        RestrictPublicBuckets: true
  DataBucketUS:
    Type: "AWS::S3::Bucket"
    Properties:
      BucketName: "data-residency-us"
      PublicAccessBlockConfiguration:
        BlockPublicAcls: true
        BlockPublicPolicy: true
        IgnorePublicAcls: true
        RestrictPublicBuckets: true
```

**Flow:**

1. Root Organization
2. Account A
3. Account B
4. Account C
5. Account D
6. Dev/Stage
7. Staging
8. Production
9. Dev/Stage
10. Staging
11. Production
12. Dev/Stage
13. Staging
14. Production
15. Dev/Stage
16. Staging
17. Production

