---
type: "practice"
difficulty: "Deep"
---

# Your experiment-tracking service and training jobs run in different cloud accounts. How would you design cross-account access so runs log metrics without long-lived keys?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Instance Roles and Access](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Instance_Roles_and_Access.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that your experiment-tracking service can log metrics from training jobs running in a separate cloud account, we need to design a secure and scalable solution that avoids the use of long-lived credentials such as access keys or secrets. The key idea is to leverage IAM roles and temporary security tokens for cross-account access.

#### Triage
What's Being Probed

The question is probing your understanding of AWS Cloud Architecture, specifically around Secure Access Between Accounts, IAM Policies, and STS (Security Token Service). It also tests your ability to design a robust system that minimizes the risk of credential exposure while ensuring seamless communication between different cloud environments.

Step-by-Step Solution
Identify the Need for Cross-Account Access: The experiment-tracking service in one account needs to log metrics from training jobs running in another account. This requires setting up a secure and temporary method for authentication.
Use AWS IAM Roles for Service Accounts: Create an IAM role with minimal permissions in the training job account that allows it to assume an IAM role in the tracking service account.
STS (Security Token Service) for Temporary Credentials: Use STS to generate temporary security credentials based on the assumed IAM role in the tracking service account.
Secure Communication Over HTTPS: Ensure all communication between the training jobs and the experiment-tracking service is over HTTPS to protect data in transit.

#### First-Principles Solution
1. Setup IAM Roles

In the Training Job Account:

Create an IAM role with a policy that allows it to assume another role (e.g., AssumeRolePolicy).
JSON
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Service": "ec2.amazonaws.com"
            },
            "Action": "sts:AssumeRole",
            "Condition": {}
        }
    ]
}

In the Experiment Tracking Service Account:

Create an IAM role with a policy that grants permissions to log metrics (e.g., metricsWriter).
JSON
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": ["logs:CreateLogGroup", "logs:CreateLogStream", "logs:PutLogEvents"],
            "Resource": "*"
        }
    ]
}
2. Use STS for Temporary Credentials

In the Training Job Account, create an instance profile or role that can assume the AssumeRolePolicy to get temporary credentials from STS in the tracking service account.

PYTHON
import boto3

sts_client = boto3.client('sts')

response = sts_client.assume_role(
    RoleArn="arn:aws:iam::<tracking-service-account-id>:role/metricsWriter",
    RoleSessionName="metrics-session"
)

temp_credentials = response['Credentials']
3. Secure Communication

Ensure the experiment-tracking service and training jobs communicate over HTTPS to protect data in transit.

#### Closing Summary

By using IAM roles for service accounts, STS for generating temporary credentials, and ensuring secure communication, we can achieve cross-account access without exposing long-lived keys or secrets. This design is scalable, secure, and minimizes the risk of credential exposure.

#### Common Mistakes
and Follow-Ups
Mistake 1: Using long-term AWS Access Keys in both accounts. This increases the risk of key exposure and should be avoided.
Mistake 2: Failing to use HTTPS for communication between services, which can lead to data breaches.

#### Follow-Up
Questions:
How would you handle auditing and logging of these cross-account operations?
What if one account needs to access multiple accounts in the network? How would you manage that?
Are there any additional security measures you might consider implementing?

**Comparison:**

| Role ARN         | Policy Statements                                       |
|------------------|--------------------------------------------------------|
| Training Job     | Allow `ec2.amazonaws.com` to assume `metricsWriter` role |
| Experiment Track | Grant permissions for logging metrics                  |

**Code:**

```python
import boto3

sts_client = boto3.client('sts')

response = sts_client.assume_role(
    RoleArn="arn:aws:iam::<tracking-service-account-id>:role/metricsWriter",
    RoleSessionName="metrics-session"
)

temp_credentials = response['Credentials']
```

**Example input / output:**

```text
Input (Training Job):
AssumeRolePolicyArn = arn:aws:iam::<tracking-service-account-id>:role/metricsWriter
SessionName = metrics-session

Output:
TempCredentials = {
    "AccessKeyId": "ABCD1234567890",
    "SecretAccessKey": "ABCDEFGHIJKLMNOPQRSTUVWXYZ1234567890",
    "Token": "abcdefghijklmnopqrstuvwxyz1234567890=.",
    "Expiration": "2023-10-10T14:00:00Z"
}
```

**Data:**

```json
{
    "TrainingJobAccountPolicy": {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Principal": {
                    "Service": "ec2.amazonaws.com"
                },
                "Action": "sts:AssumeRole",
                "Condition": {}
            }
        ]
    },
    "ExperimentTrackingServiceAccountPolicy": {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Action": ["logs:CreateLogGroup", "logs:CreateLogStream", "logs:PutLogEvents"],
                "Resource": "*"
            }
        ]
    }
}
```

**Flow:**

1. Training Job
2. Assume Role
3. Generate TemporaryCredentials
4. Send Metrics Over HTTPS
5. Experiment Tracking Service

