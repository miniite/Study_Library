---
type: "practice"
difficulty: "Expert"
---

# How does temporary credential rotation via an instance role actually work when your app makes an API call?

**Difficulty:** Expert · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Least Privilege IAM](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Least_Privilege_IAM.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with cloud fundamentals, understanding how temporary credentials are rotated through instance roles is crucial, especially in scenarios where applications make API calls. At its core, this process leverages AWS IAM roles to securely manage access tokens and ensure that application instances can perform actions on behalf of their users or other services. In this explanation, I'll break down the mechanism from first principles, highlighting key steps and common pitfalls.

Firstly, it's important to recognize that instance roles in AWS are part of a broader strategy for managing permissions in a secure manner. When an EC2 instance is launched with an IAM role attached, the role provides temporary security credentials to the instance. These credentials are used by the application running on the instance to make API calls.

Here’s how it works step-by-step:

EC2 Instance Launch: The EC2 instance is launched and configured with a specific IAM role.
Assume Role Request: When an application needs to make an API call, it requests temporary credentials from AWS STS (Security Token Service).
STS Response: The STS service verifies the request and issues temporary security credentials based on the policies attached to the instance role.
Token Refresh Mechanism: Credentials are short-lived, typically ranging from 1 to 6 hours. When an application needs to make a new API call, it renews these tokens by making another request to STS.

Now, let's dive into each step with more detail:

EC2 Instance Launch:

The instance is created and an IAM role is attached via the EC2 launch configuration or user data script.
This role defines the permissions that can be assumed by the application running on the instance. For example, the policy might grant access to S3 buckets, Lambda functions, etc.

Assume Role Request:

The application, typically a service running inside the EC2 instance, sends an assume-role request to AWS STS.
This request includes details such as the ARN of the role and optionally, session tags (which can be used for additional context).

STS Response:

AWS STS verifies that the request is authorized based on the policies attached to the IAM role.
It then returns temporary security credentials in a JSON Web Token (JWT) format. These include an access key ID, secret access key, and session token.

Token Refresh Mechanism:

The application stores these temporary credentials securely within its environment or as environment variables.
When the application makes a new API call, it uses these stored credentials to authenticate itself with AWS services.
To ensure ongoing security, the application periodically renews the tokens by sending another assume-role request. This is done to prevent unauthorized access and maintain compliance.

#### Common Mistakes
and Follow-Ups
Storing Credentials Securely: Applications must securely store and handle credentials, avoiding hardcoding them directly in code or configuration files.
Token Expiration Handling: Developers should implement robust mechanisms for token renewal, ensuring that API calls are not interrupted due to expired tokens.
Security Policies: Ensuring the IAM role policies are correctly configured is critical. Overly permissive policies can lead to security vulnerabilities.

In summary, temporary credential rotation via an instance role in AWS is a secure and dynamic method of managing access within cloud environments. By understanding the steps involved, developers can ensure their applications operate efficiently while maintaining high levels of security.

**Comparison:**

```
Step	Description
1	EC2 instance is launched with an IAM role.
2	Application requests temporary credentials from AWS STS.
3	AWS STS issues temporary security credentials.
4	Credentials are refreshed periodically to ensure ongoing access.
```

**Code:**

```python
# Example Python code for assuming a role using Boto3

import boto3

sts_client = boto3.client('sts')

response = sts_client.assume_role(
    RoleArn="arn:aws:iam::123456789012:role/ExampleRole",
    RoleSessionName="AssumeRoleSession"
)

temp_credentials = response['Credentials']
print("Access Key:", temp_credentials['AccessKeyId'])
print("Secret Access Key:", temp_credentials['SecretAccessKey'])
print("Session Token:", temp_credentials['SessionToken'])
```

**Example input / output:**

```text
Input:
- Requesting temporary credentials from STS with role ARN and session name.

Output:
- JSON response containing access key ID, secret access key, and session token.
```

**Data:**

```json
{
  "roleArn": "arn:aws:iam::123456789012:role/ExampleRole",
  "roleSessionName": "AssumeRoleSession"
}
```

**Flow:**

1. Yes
2. No
3. EC2 Instance Launch
4. Attach IAM Role
5. STS Assume Role Request
6. STS Response (Credentials)
7. Store Credentials Securely
8. API Call with StoredCredentials
9. Tokens Expired?
10. Refresh Tokens via STS
11. Continue API Calls

