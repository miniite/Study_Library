---
type: "practice"
difficulty: "Deep"
---

# When a business unit demands GCP for its ML tooling while your platform standard is AWS, how do you draw the line between a sanctioned second cloud and shadow IT you must shut down?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Cross-cloud Data Contracts](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Multi_Cloud_Data/Cross-cloud_Data_Contracts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When a business unit requests Google Cloud Platform (GCP) for their machine learning tooling while the company's standard platform is AWS, the key challenge lies in balancing compliance with organizational standards and fostering innovation. This scenario involves drawing a clear line between sanctioned secondary clouds that align with corporate policies and shadow IT that could pose security or operational risks.

#### Triage

The question essentially asks how to manage a situation where there are conflicting technical standards within an organization. The primary concerns here are:

Compliance: Ensuring that the chosen cloud environment adheres to organizational policies.
Risk Management: Evaluating and mitigating potential security, compliance, and operational risks associated with using GCP instead of AWS.
Innovation and Efficiency: Supporting business units in their technical requirements while maintaining a cohesive infrastructure.

#### Assumptions
The organization has established cloud standards around AWS for various reasons (cost, compliance, ecosystem integration).
There is a need to balance these standards with the flexibility required by specific business units.
The goal is not just to say "yes" or "no," but to find an optimal solution that supports innovation without compromising security and compliance.

#### Solution
Evaluate Business Needs: Conduct a detailed analysis of the business unit's requirements, including data size, performance needs, regulatory constraints, and budget considerations.
Risk Assessment: Perform a thorough risk assessment for using GCP, considering factors such as integration with existing AWS-based systems, data privacy laws (e.g., GDPR), and potential security vulnerabilities.
Pilot Program: Propose a pilot program to test the use of GCP for ML tooling on a small scale. This will help identify any issues early and build a case for broader adoption if successful.
Policy and Governance Frameworks: Develop or update existing policy frameworks to include GCP as a sanctioned cloud environment while maintaining strict governance controls.
Hybrid Architecture Consideration: Explore hybrid architectures that allow seamless data transfer between AWS and GCP, ensuring compliance without requiring business units to choose one over the other entirely.

#### Implementation
Steps
Define Objectives: Clearly outline what success looks like for both short-term pilot objectives and long-term strategic goals.
Collaborate with Stakeholders: Engage key stakeholders from IT, legal, finance, and HR departments to gain buy-in and ensure alignment.
Choose the Right Tools: Select tools and services that support both AWS and GCP environments, such as multi-cloud management platforms or data integration solutions.
Monitor and Report: Establish a monitoring framework to track the performance and security of GCP usage and provide regular reports to stakeholders.

#### Closing Summary

Drawing the line between sanctioned secondary clouds like GCP and shadow IT involves a strategic approach that balances innovation with compliance and risk management. By carefully evaluating business needs, conducting thorough risk assessments, and implementing robust governance frameworks, you can support the organization's technical requirements while maintaining security and operational integrity.

#### Common Mistakes/Follow-Ups
Ignoring Compliance: Failing to address regulatory constraints could lead to significant legal issues.
Overlooking Security Risks: Neglecting proper security measures in GCP could compromise sensitive data.
Inadequate Communication: Poor stakeholder engagement can result in resistance and delays.

**Comparison:**

```
Cloud	CPU Hours (Hourly)	Memory GB (Daily)	Data Transferred GB (Monthly)
AWS	1200	500	10,000
GCP	300	150	2,500
```

**Code:**

```text
# Example of a simple Python script for monitoring cloud usage
import boto3

def get_aws_usage():
    client = boto3.client('boto3')
    response = client.get_usage()
    return response['usage']

def get_gcp_usage():
    # Placeholder function to simulate GCP usage retrieval
    pass

# Example of integrating both AWS and GCP data
def monitor_both_clouds(aws_client, gcp_client):
    aws_usage = get_aws_usage()
    gcp_usage = get_gcp_usage()

    if gcp_usage > 0:
        print("GCP Usage Detected")
    else:
        print("No GCP Usage")

monitor_both_clouds('AWS Client', 'GCP Client')
```

**Example input / output:**

```text
Input:
- AWS Usage Data
- GCP Usage Data

Output:
GCP Usage Detected
```

**Data:**

```json
{
  "awsUsage": {
    "cpuHours": 1200,
    "memoryGB": 500,
    "dataTransferredGB": 10000
  },
  "gcpUsage": {
    "cpuHours": 300,
    "memoryGB": 150,
    "dataTransferredGB": 2500
  }
}
```

**Flow:**

1. Positive
2. Negative
3. Define Objectives
4. Evaluate Business Needs
5. Propose Pilot Program
6. Continue with AWS Standard
7. Develop Policy Framework
8. Monitor and Report

