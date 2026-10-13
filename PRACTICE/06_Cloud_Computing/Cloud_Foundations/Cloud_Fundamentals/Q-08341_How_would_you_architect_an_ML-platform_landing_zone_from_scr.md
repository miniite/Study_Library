---
type: "practice"
difficulty: "Expert"
---

# How would you architect an ML-platform landing zone from scratch for a regulated healthtech about to serve models in five countries, so accelerator governance, data residency, and account structure are correct on day one rather than retrofitted?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Data Lifecycle and Retention](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Stateless_by_Default/Data_Lifecycle_and_Retention.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To architect an ML platform landing zone for a regulated healthtech company that will soon be serving models in five countries, we need to ensure compliance with governance, data residency requirements, and account structure from day one. This involves setting up a robust cloud infrastructure that adheres to regulatory standards, manages multi-region data flow efficiently, and supports agile development practices.

First, let's break down the key components:

Governance: Ensuring all models are built and deployed within compliant environments.
Data Residency: Storing and processing data in accordance with local laws and regulations.
Account Structure: Organizing cloud resources into logical units that align with business and regulatory needs.

#### Triage

The question is probing your understanding of cloud fundamentals, including multi-region deployment, governance best practices, and resource management. Key assumptions include the need for compliance with international data protection regulations (e.g., GDPR) and adherence to best practices for secure model deployment.

#### First-Principles Solution

Multi-Region Architecture:

Set up a landing zone in each of the five countries where the models will be deployed.
Use AWS Regions or Google Cloud Zones that align with local data residency requirements.

Governance and Compliance:

Implement strict governance policies using services like AWS Control Tower for AWS, or Anthos Governance for Google Cloud.
Establish a centralized logging and monitoring system to track model performance and compliance across regions.
Use role-based access control (RBAC) to ensure that only authorized personnel have access to sensitive data.

Data Residency:

Store training and production datasets in local AWS or GCP storage services within each country’s region.
Implement data encryption at rest and in transit using KMS for AWS or Cloud Key Management Service (KMS) for GCP.
Ensure all model artifacts are stored locally to meet data residency requirements.

Account Structure:

Organize cloud resources into separate accounts per country, ensuring isolation of sensitive data and operations.
Use shared services account(s) for common infrastructure like VPCs, IAM roles, and logging solutions.
Implement cross-account access policies using AWS Cross-Account Roles or Google Cloud Shared VPC.
Detailed Solution

Cloud Provider Selection:

For this example, let’s use AWS as the primary provider due to its extensive compliance offerings for healthtech.
Select regions such as EU (Frankfurt), US East (N. Virginia), Asia Pacific (Tokyo), Latin America (São Paulo), and Australia (Sydney).

Landing Zone Setup:

Create a central landing zone in a primary region, say AWS GovCloud for enhanced security.
Deploy a set of common services here: VPC peering, centralized logging, monitoring stack using CloudWatch.

Country-Specific Landing Zones:

In each country’s region, create a separate landing zone with its own set of resources.
Use AWS Organizations or Google Cloud Resource Manager to manage these accounts centrally while keeping them isolated.

Data Management:

Set up S3 buckets in each region for storing raw and processed data.
Use Glue or BigQuery for ETL jobs, ensuring all operations are logged and monitored.
Implement data lineage tracking using AWS Data Pipeline or Google Cloud Dataprep.

Model Deployment:

Deploy models using SageMaker or Vertex AI in each region.
Ensure models are tested against local regulations and compliance requirements before deployment.
Use A/B testing and canary releases to gradually roll out models without compromising patient data.

Security and Compliance:

Implement VPC peering between regions where necessary, ensuring secure communication.
Configure security groups and network ACLs to enforce isolation and access control.
Set up a comprehensive IAM policy for role-based access, with least privilege principles applied.

Monitoring and Logging:

Use CloudWatch or Stackdriver for continuous monitoring of model performance and resource utilization.
Implement a centralized logging system using ELK stack (Elasticsearch, Logstash, Kibana) or Splunk to aggregate logs from all regions.

#### Closing Summary

By following these steps, we ensure that the ML platform is compliant with regulatory requirements from day one. The architecture leverages multi-region capabilities of cloud providers while maintaining strict governance and data residency policies. Each step is designed to support agile development practices and facilitate future scaling or expansion into additional regions.

#### Common Mistakes/Follow-Ups
Mistake: Failing to account for local regulatory requirements, leading to non-compliance.

#### Follow-Up
What are the specific compliance challenges in each region? How will you address them?
Mistake: Overlooking network segmentation, which could lead to security vulnerabilities.

#### Follow-Up
Explain how you would implement VPC peering and network policies to ensure secure communication.

**Comparison:**

```
Region	S3 Bucket Name	IAM Role Name
EU	eu-fra-data-bucket	eu-fra-data-access-role
US	us-nva-data-bucket	us-nva-data-access-role
AP	ap-tok-data-bucket	ap-tok-data-access-role
LA	la-sao-data-bucket	la-sao-data-access-role
AU	au-syd-data-bucket	au-syd-data-access-role
```

**Code:**

```python
# Example of setting up a basic logging pipeline in AWS using CloudWatch Logs
import boto3

def setup_cloudwatch_logs(region):
    client = boto3.client('logs', region_name=region)
    response = client.create_log_group(logGroupName='/aws/lambda/ml-platform')
    print(f"Log group created: {response['logGroupName']}")

setup_cloudwatch_logs('eu-central-1')
```

**Example input / output:**

```text
Input:
- Regional compliance requirements for each country.

Output:
- Centralized logging and monitoring system setup.
- Country-specific landing zones with VPC peering configurations.
- IAM roles and policies defined to enforce least privilege access.
- Data residency and encryption mechanisms implemented in S3 buckets.
```

**Data:**

```json
{
  "data_landing_zone": {
    "eu_frankfurt": {
      "s3_bucket": "eu-fra-data-bucket",
      "iam_role": "eu-fra-data-access-role"
    },
    "us_north_virginia": {
      "s3_bucket": "us-nva-data-bucket",
      "iam_role": "us-nva-data-access-role"
    },
    "ap_tokyoo": {
      "s3_bucket": "ap-tok-data-bucket",
      "iam_role": "ap-tok-data-access-role"
    },
    "la_sao_paulo": {
      "s3_bucket": "la-sao-data-bucket",
      "iam_role": "la-sao-data-access-role"
    },
    "au_sydney": {
      "s3_bucket": "au-syd-data-bucket",
      "iam_role": "au-syd-data-access-role"
    }
  }
}
```

**Flow:**

1. Yes
2. No
3. Data Ingest
4. S3 Buckets
5. Is Data Localized?
6. (Local) ETL & ML
7. Cross-Region Transfer
8. (Country-Specific) Models
9. Cross-Region Transfer
10. Centralized Monitoring
11. Compliance Logging

