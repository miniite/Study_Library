---
type: "practice"
difficulty: "Expert"
---

# Explain how you'd set up cross-account/cross-cloud data sharing of Gold to external partners without copying data, and what the security and cost implications are.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Controlling Gold Sprawl](<../../../../02_Data_Engineering/Storage_and_Lakehouse/Medallion_Architecture/Controlling_Gold_Sprawl.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of setting up cross-account/cross-cloud data sharing of Gold-level data with external partners without duplicating it, we'll leverage AWS services such as Data Exchange, S3 VPC Endpoints, and Identity and Access Management (IAM) to ensure secure and efficient data access. The goal is to provide read-only access while minimizing costs and maintaining strict security controls.

#### Triage
Understanding the Question

The core of this question revolves around enabling secure and efficient cross-account/cross-cloud data sharing for Gold-level data without duplicating it. This involves understanding the security requirements, identifying suitable AWS services, and crafting a solution that balances security with cost efficiency.

#### First-Principles Solution

Data Sourcing and Storage:

Ensure all Gold data is stored in a highly secure environment, such as an Amazon S3 bucket within a VPC.
Use encryption at rest (AWS KMS) to protect the data.

Identity and Access Management (IAM):

Create IAM roles with least privilege access for external partners.
Grant read-only permissions using AWS Policies or IAM Roles.

Cross-Account Sharing:

Utilize AWS Data Exchange to share datasets across accounts securely.
Leverage S3 VPC Endpoints to enable private and secure data transfer between accounts.

Monitoring and Logging:

Set up CloudWatch for monitoring access attempts and suspicious activities.
Use GuardDuty for threat detection and compliance checks.

Cost Management:

Monitor AWS spend using Cost Explorer to ensure cost control.
Implement budget alerts via AWS Budgets to prevent overspending.
Detailed Steps

Data Storage and Encryption:

BASH
# Create an S3 bucket within a VPC for secure storage
aws s3api create-bucket --bucket <gold-data-bucket> \
  --region us-west-2 \
  --create-bucket-configuration LocationConstraint=us-west-2
Enable server-side encryption with AWS KMS:
BASH
aws s3api put-bucket-cors --bucket <gold-data-bucket> \
  --cors-configuration file://cors.json

IAM Role for External Partners:

Create an IAM role in the provider account that allows read-only access to S3 buckets.
JSON
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "",
      "Effect": "Allow",
      "Action": ["s3:GetObject"],
      "Resource": ["arn:aws:s3:::<gold-data-bucket>/*"]
    }
  ]
}

Data Exchange Setup:

Publish the dataset on AWS Data Exchange.
BASH
aws dataexchange create-asset --name "<Gold-Dataset>" \
  --description "High-quality Gold-level data" \
  --format Type=CSV,Compressed=false \
  --sourceDetails.file s3://<gold-data-bucket>/<file-path>

Cross-Account Access:

Set up an IAM role in the consumer account with read-only access to S3.
BASH
aws iam create-role --role-name <consumer-access-role> \
  --assume-role-policy-document file://iam-policy.json
Attach the required policy to allow S3 read access.

Monitoring and Logging:

Enable CloudWatch logs for the IAM role.
BASH
aws logs create-log-group --log-group-name <gold-data-access-logs>

Cost Management:

Set up AWS Budgets for cost monitoring.
BASH
aws budget create-budget \
  --budget file://budget.json \
  --account-id <consumer-account-id>

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, setting up cross-account/cross-cloud data sharing of Gold-level data requires a robust approach involving secure storage, IAM policies, Data Exchange for metadata management, VPC endpoints for private access, monitoring tools, and cost controls. Key mistakes to avoid include overly permissive IAM policies, lack of encryption, insufficient monitoring, and poor budget planning.

#### Follow-up
questions might involve discussing specific use cases, integration with other AWS services like Glue for data processing, or additional security measures such as network isolation and audit trails.

**Comparison:**

```
Component	Description
S3 Bucket	Secure storage for Gold-level data.
IAM Role	Grants read-only access to external partners.
Data Exchange	Metadata management and sharing across accounts.
VPC Endpoint	Private network connection between accounts for secure data transfer.
CloudWatch Logs	Monitoring of access attempts and suspicious activities.
AWS Budgets	Cost control and budget alerts.
```

**Code:**

```
Example S3 bucket creation and CORS configuration

aws s3api create-bucket --bucket <gold-data-bucket>
--region us-west-2
--create-bucket-configuration LocationConstraint=us-west-2

aws s3api put-bucket-cors --bucket <gold-data-bucket>
--cors-configuration file://cors.json

CODE
Copy

```json
# IAM Policy for read-only S3 access
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "",
      "Effect": "Allow",
      "Action": ["s3:GetObject"],
      "Resource": ["arn:aws:s3:::<gold-data-bucket>/*"]
    }
  ]
}
JSON
Copy
# AWS Budget JSON example
{
  "Budget": {
    "BudgetName": "<Gold-Data-Budget>",
    "AccountName": "<Consumer-Account-Name>",
    "TimePeriod": {
      "Start": "2023-10-01T00:00:00Z",
      "End": "2024-09-30T23:59:59Z"
    },
    "BudgetLimit": {
      "Amount": 500.0,
      "Unit": "USD"
    },
    "ThresholdPercentage": 80
  }
}
```

**Example input / output:**

```text
Input:
- Gold-level data stored in S3 bucket within VPC.
- IAM roles with least privilege access created for external partners.

Output:
- External partners can access Gold data using AWS Data Exchange.
- Secure and private transfer via S3 VPC Endpoints.
- Cost management through AWS Budgets.
```

**Data:**

```json
{
  "goldDataBucket": "<gold-data-bucket>",
  "consumerAccessRole": "<consumer-access-role>",
  "awsDataExchangeAssetName": "<Gold-Dataset>",
  "awsBudgetName": "<Gold-Data-Budget>",
  "accountID": <consumer-account-id>
}
```

**Flow:**

1. Data Sourcing and Storage
2. S3 Bucket with Encryption
3. IAM Role for External Partners
4. Data Exchange Setup
5. VPC Endpoint for PrivateAccess
6. Monitoring and Logging(CloudWatch)
7. Cost Management (AWSBudgets)

