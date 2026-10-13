---
type: "practice"
difficulty: "Deep"
---

# You need to give a data-science team read access to production data that spans two clouds without copying it into notebooks — design the governed access pattern and its audit trail.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Cross-cloud Data Contracts](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Multi_Cloud_Data/Cross-cloud_Data_Contracts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, our goal is to provide a data science team with read-only access to production data stored in multiple cloud environments without copying the data into notebooks or other local storage. This involves designing a governed and audited access pattern that ensures security, compliance, and traceability of data usage. The key idea here is to leverage managed services and policies provided by cloud providers while implementing additional controls through a centralized governance framework.

First, we need to understand what "read-only" means in this context: it implies that the team should be able to query or extract insights from the data without modifying it. The access must also comply with organizational security standards and regulations, such as GDPR or HIPAA.

To achieve this, we will use a combination of cloud-specific features like AWS Glue Data Catalog and Azure Synapse Analytics, along with an external identity and access management (IAM) service to manage user roles and permissions. We'll implement role-based access control (RBAC), data masking, and encryption to ensure that only authorized users can read the data.

#### Triaging the Question

The interviewer is probing your understanding of multi-cloud architecture design principles, particularly focusing on:

Data Governance: How you enforce data policies and controls across multiple cloud environments.
Access Management: Implementing fine-grained access control to ensure that data science team members have only necessary permissions.
Audit Trails: Mechanisms for tracking who accessed the data and when.
Security Compliance: Ensuring that all access patterns meet organizational security standards.

#### First-Principles Solution

Cloud Data Catalogs:

Use AWS Glue Data Catalog or Azure Data Catalog to manage metadata about the production datasets across both clouds.
Ensure that the catalog is updated in real-time with any changes to the underlying data stores.

Data Masking and Anonymization:

Apply data masking policies to sensitive fields, such as personal identifiers, to protect privacy while allowing analysis.
Use tools like AWS Glue Data Transformer or Azure Purview for metadata management and automated data governance.

Role-Based Access Control (RBAC):

Implement RBAC in both cloud environments using IAM roles specifically tailored for the data science team.
Create a role that allows read-only access to specific datasets, ensuring that no write operations are possible.

Centralized Identity and Access Management:

Use an external IAM service like AWS IAM or Azure AD B2B to manage user identities and their roles across both clouds.
Implement single sign-on (SSO) for seamless authentication and authorization.

Data Lake and Data Warehouse Integration:

Ensure that the data science team can query production data directly from cloud-native services using SQL-based interfaces like AWS Athena or Azure Synapse Analytics.
Use managed services to reduce the burden of managing and maintaining infrastructure, allowing the team to focus on analysis rather than operational tasks.

Audit Trails and Monitoring:

Set up detailed logging and monitoring for all data access activities in both clouds.
Integrate with cloud-native security tools like AWS CloudTrail or Azure Monitor to track every read operation.
Implement alerting mechanisms that notify relevant stakeholders of any suspicious activity or breaches.

Compliance and Regulatory Monitoring:

Ensure compliance by regularly auditing data usage against organizational policies and regulatory requirements.
Use automated checks provided by cloud providers, such as AWS Config or Azure Policy, to enforce compliance rules.

#### Closing Summary
+ Common Mistakes/Follow-Ups

The key takeaway is that a robust multi-cloud data architecture requires a combination of managed services, RBAC, centralized identity management, and comprehensive auditing. The implementation must be scalable and secure, with the ability to adapt to changing regulatory requirements. Common mistakes include over-relying on local copies of data or neglecting detailed logging and monitoring. Always ensure that your solution is aligned with organizational security best practices.

#### Closing Summary

In conclusion, designing a governed access pattern for a data-science team in a multi-cloud environment involves implementing role-based access control, leveraging cloud-native services for data cataloging and querying, and establishing robust audit trails to track data usage. This approach ensures that the team can analyze production data while maintaining strict security and compliance.

#### Common Mistakes/Follow-Ups
Local Copies: Avoid copying data into notebooks or other local storage.
Insufficient Auditing: Ensure comprehensive logging and monitoring of all data access activities.
Lack of Compliance Monitoring: Regularly audit usage against organizational policies and regulatory requirements.
Overcomplicated Governance: Keep the solution simple and aligned with existing best practices.

**Comparison:**

```
Resource	Action	Effect
S3	GetObject, ListBucket	Allow
Data Lake Store	Read	Allow
```

**Code:**

```python
# Example code for creating a read-only role in AWS IAM using Boto3
import boto3

def create_read_only_role(role_name, policy_arn):
    iam = boto3.client('iam')
    response = iam.create_role(
        RoleName=role_name,
        AssumeRolePolicyDocument='{"Version": "2012-10-17", "Statement": [{"Action": "sts:AssumeRole", "Effect": "Allow"}]}'
    )

    # Attach policy for read-only access
    iam.attach_role_policy(
        RoleName=role_name,
        PolicyArn=policy_arn
    )

create_read_only_role('DataScienceReadOnly', 'arn:aws:iam::123456789012:policy/ReadOnlyAccess')
```

**Example input / output:**

```text
Input:
- Cloud environment A: AWS S3 bucket named 'production-data-bucket'
- Cloud environment B: Azure Data Lake Store named 'data-lake-store'

Output:
- DataScienceReadOnly IAM role created with permission to read from both S3 and ADLS.
- Real-time metadata catalog updated in AWS Glue and Azure Synapse Analytics.

Dry-run:
1. User logs into the cloud console using their AWS or Azure account.
2. User has no direct access to production data stores but can query them via managed services like Athena or Synapse.
```

**Data:**

```json
{
  "data_catalog": {
    "name": "ProductionDataCatalog",
    "description": "Central metadata store for production data spanning multiple clouds."
  },
  "iam_policy": {
    "Name": "ReadOnlyAccess",
    "PolicyDocument": {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Effect": "Allow",
          "Action": ["s3:GetObject", "s3:ListBucket"],
          "Resource": ["arn:aws:s3:::production-data-bucket/*"]
        }
      ]
    }
  }
}
```

**Flow:**

1. User Authentication
2. Central IAM Service (AWS ADor Azure AD)
3. RBAC Role Assignment
4. Role-Based Access to CloudServices
5. Data Lake and DataWarehouse Queries
6. Audit Logging & Monitoring

